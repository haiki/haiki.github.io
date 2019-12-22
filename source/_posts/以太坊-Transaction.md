---
title: 以太坊-Transaction
date: 2019-12-22 10:55:27
categories:
- 区块链
tags:
- 以太坊
- transaction
---

- 本文主要参考（并根据自己的理解进行思路的梳理）
  - [以太坊交易源码分析](https://blog.csdn.net/TurkeyCock/article/details/80485391)
  - [GoLand导入并编译以太坊源码go-ethereum](https://blog.csdn.net/yzpbright/article/details/83538362s)
  - [交易签名以及哈希值的计算](https://ethereum.iethpay.com/how-to-sign-and-hash-tx.html)

## 交易定义

- 由 `EOA（externally owned account）`生成的，签过名的消息
- 万事始于交易。交易可触发以太坊（全局的单个状态机）状态的转换，调用智能合约在 `EVM` 中的执行

## 交易基本流程

- 发起交易：指定目标地址和交易金额，以及需要的 gas/gaslimit，还可以填充 data 字段

- 交易签名：使用账户私钥对交易（实际是交易的 hash）进行签名

- 提交交易：把交易加入到交易缓冲池 txpool 中（会先对交易签名进行验证）

- 广播交易：通知 EVM 执行，同时把交易信息广播给其他结点

![transaction-execute.jpg](https://i.loli.net/2019/12/22/MBHsouZ8PK6AGip.png)

### 交易处理入口函数

用户通过 `JSON RPC` 发起 `eth_sendTransaction` 请求，最终会调用 `PublicTransactionPoolAPI` 的实现，代码位于` internal/ethapi/api.go`。

首先根据 `from` 地址查找到对应的 `wallet`。根据参数值去设置默认值，比如 `amount` `gasPrice`  `nonce` 字段可以没有输入。还会检查 `data` 和 `input` 是否重复输入，并且不相等。对参数值进行检查，如果是创建合约，检查 `data`是否为空。

接着主要做了以下3件事：

- 通过 `SendTxArgs.toTransaction() `创建交易

- 通过`Wallet.SignTx()`对交易进行签名。Wallet是一个接口，具体实现在 keyStoreWallet 中。
- 通过`submitTransaction()`提交交易

```go
// SendTransaction creates a transaction for the given argument, sign it and submit it to the
// transaction pool.
func (s *PublicTransactionPoolAPI) SendTransaction(ctx context.Context, args SendTxArgs) (common.Hash, error) {
	// Look up the wallet containing the requested signer
	account := accounts.Account{Address: args.From}

	wallet, err := s.b.AccountManager().Find(account)
	if err != nil {
		return common.Hash{}, err
	}

	if args.Nonce == nil {
		// Hold the addresse's mutex around signing to prevent concurrent assignment of the same nonce to multiple accounts.
		s.nonceLock.LockAddr(args.From)
		defer s.nonceLock.UnlockAddr(args.From)
	}

	// Set some sanity defaults and terminate on failure
	if err := args.setDefaults(ctx, s.b); err != nil {
		return common.Hash{}, err
	}
	// Assemble the transaction and sign with the wallet
	tx := args.toTransaction() //*****

	signed, err := wallet.SignTx(account, tx, s.b.ChainConfig().ChainID)//*****
	if err != nil {
		return common.Hash{}, err
	}
	return SubmitTransaction(ctx, s.b, signed)//*****
}
```

### 创建交易实例

首先处理输入参数的 `input/data` 字段，推荐用 `input`，`data` 是为了向后兼容。

如果目标地址为空的话，表示这是一个创建智能合约的交易，调用 `NewContractCreation()`。否则说明这是一个普通交易，调用 `NewTransaction()`。不管调用哪个，最终都会生成一个Transaction实例，处理函数是一样的，只是 `to` 参数有没有值的区别，transaction 类型字段见下。

```go
//代码位于internal/ethapi/api.go
func (args *SendTxArgs) toTransaction() *types.Transaction {
	var input []byte
	if args.Input != nil {
		input = *args.Input
	} else if args.Data != nil {
		input = *args.Data
	}
	if args.To == nil {
		return types.NewContractCreation(uint64(*args.Nonce), (*big.Int)(args.Value), uint64(*args.Gas), (*big.Int)(args.GasPrice), input)
	}
	return types.NewTransaction(uint64(*args.Nonce), *args.To, (*big.Int)(args.Value), uint64(*args.Gas), (*big.Int)(args.GasPrice), input)
}
```

#### SendTxArgs 结构

和 `JSON` 字段相应的，包括了地址、gas、金额这些交易信息，`nonce` 是一个随账户交易次数自增的数字，一般会在默认设置中自动获取填充。交易还可以携带一些额外数据，存放在 `data` **或者** `input` 字段中。

```go
//代码位于internal/ethapi/api.go
// SendTxArgs represents the arguments to sumbit a new transaction into the transaction pool.
type SendTxArgs struct {
	From     common.Address  `json:"from"`
	To       *common.Address `json:"to"`
	Gas      *hexutil.Uint64 `json:"gas"`
	GasPrice *hexutil.Big    `json:"gasPrice"`
	Value    *hexutil.Big    `json:"value"`
	Nonce    *hexutil.Uint64 `json:"nonce"`
	// We accept "data" and "input" for backwards-compatibility reasons. "input" is the newer name and should be preferred by clients.
	Data  *hexutil.Bytes `json:"data"`
	Input *hexutil.Bytes `json:"input"`
}
```

#### transaction类型

主要就是包含了一个 `txdata` 类型的字段，其他3个都是缓存。

```go
// 代码位于 core/types/transaction.go
type Transaction struct {
	data txdata
	// caches
	hash atomic.Value
	size atomic.Value
	from atomic.Value
}
```

#### txdata 类型

相对于输入参数中的字段，多了 `V、R、S` 和 `hash` 4 个字段，少了 `from` 字段。<v,r,s>是签名后才能得到的，现在生成的交易实例中不包含。

 其他节点收到交易时，通过 <v,r,s> 及 消息hash 计算出 `from` 字段，参见 **交易签名** 部分。

```go
// 代码位于 core/types/transaction.go
type txdata struct {
	AccountNonce uint64          `json:"nonce"    gencodec:"required"`
	Price        *big.Int        `json:"gasPrice" gencodec:"required"`
	GasLimit     uint64          `json:"gas"      gencodec:"required"`
	Recipient    *common.Address `json:"to"       rlp:"nil"` // nil means contract creation
	Amount       *big.Int        `json:"value"    gencodec:"required"`
	Payload      []byte          `json:"input"    gencodec:"required"`

	// Signature values
	V *big.Int `json:"v" gencodec:"required"`
	R *big.Int `json:"r" gencodec:"required"`
	S *big.Int `json:"s" gencodec:"required"`

	// This is only used when marshaling to JSON.
	Hash *common.Hash `json:"hash" rlp:"-"`
}
```

### 交易签名

先通过 `Keccak-256` 算法计算交易数据的 has h值，然后结合账户的私钥，通过 ECDSA（Elliptic Curve Digital Signature Algorithm），也就是椭圆曲线数字签名算法生成签名数据。



![SignTx.jpg](https://i.loli.net/2019/12/22/X5xb6N4hMg1yDGW.png)



```go
//代码位于 accounts/keystore/wallet.go
/* SignTx implements accounts.Wallet, attempting to sign the given transaction
 with the given account. If the wallet does not wrap this particular account,
 an error is returned to avoid account leakage (even though in theory we may
/be able to sign via our shared keystore backend).*/
func (w *keystoreWallet) SignTx(account accounts.Account, tx *types.Transaction, chainID *big.Int) (*types.Transaction, error) {
	// Make sure the requested account is contained within
	if !w.Contains(account) {
		return nil, accounts.ErrUnknownAccount
	}
	// Account seems valid, request the keystore to sign
	return w.keystore.SignTx(account, tx, chainID) //*****
}
```

插播一条：从交易原信息和<v,r,s>生成 `from` 字段的示意图。TODO：代码示例

![txCalFrom.jpg](https://i.loli.net/2019/12/22/iaUXbWy45P91R6O.png)



#### keystore.SignTx

这里会首先判断账户是否已经解锁，如果已经解锁的话就可以获取它的私钥。

然后创建签名器，如果要符合 EIP155 规范的话，需要把 chainID 传进去，也就是我们的 “--networkid” 令行参数。

调用一个全局函数 **SignTx()** 完成签名.

```go
//代码位于 accounts/keystore/keystore.go
//SignTx signs the given transaction with the requested account.
func (ks *KeyStore) SignTx(a accounts.Account, tx *types.Transaction, chainID *big.Int) (*types.Transaction, error) {
	// Look up the key to sign with and abort if it cannot be found
	ks.mu.RLock()
	defer ks.mu.RUnlock()

	unlockedKey, found := ks.unlocked[a.Address]
	if !found {
		return nil, ErrLocked
	}
	// Depending on the presence of the chain ID, sign with EIP155 or homestead
	if chainID != nil {
		return types.SignTx(tx, types.NewEIP155Signer(chainID), unlockedKey.PrivateKey)
	}
	return types.SignTx(tx, types.HomesteadSigner{}, unlockedKey.PrivateKey)
}
```



#### 全局 types.signTx 完成签名

主要分为3个步骤：

- 生成交易的hash值
- 根据hash值和私钥生成签名
- 把签名数据填充到Transaction实例中

```go
//代码位于 core/types/transaction_signing.go
// SignTx signs the transaction using the given signer and private key
func SignTx(tx *Transaction, s Signer, prv *ecdsa.PrivateKey) (*Transaction, error) {
	h := s.Hash(tx)
	sig, err := crypto.Sign(h[:], prv)
	if err != nil {
		return nil, err
	}
	return tx.WithSignature(s, sig)
}
```

##### 交易散列生成

```go
//代码位于 core/types/transaction_signing.go
// Hash returns the hash to be signed by the sender.
// It does not uniquely identify the transaction.
func (s EIP155Signer) Hash(tx *Transaction) common.Hash {
	return rlpHash([]interface{}{
		tx.data.AccountNonce,
		tx.data.Price,
		tx.data.GasLimit,
		tx.data.Recipient,
		tx.data.Amount,
		tx.data.Payload,
		s.chainId, uint(0), uint(0),
	})
}
```
先进行RLP编码（一种数据序列化方法），然后再用SHA3-256生成hash值。TODO：RLP 编码实现。

```go
//代码位于 core/types/block.go
func rlpHash(x interface{}) (h common.Hash) {
	hw := sha3.NewLegacyKeccak256()
	rlp.Encode(hw, x)
	hw.Sum(h[:0])
	return h
}
```

##### 根据hash值和私钥生成签名

TODO：签名的过程

通过 ECDSA 算法生成签名数据，最终会返回的签名是一个字节数组，按R / S / V的顺序排列。

```go
//代码位于 crypto/signature_cgo.go
/*Sign calculates an ECDSA signature.
 This function is susceptible to chosen plaintext attacks that can leak
 information about the private key that is used for signing. Callers must
be aware that the given digest cannot be chosen by an adversery. Common
solution is to hash any input before calculating the signature.*/

// The produced signature is in the [R || S || V] format where V is 0 or 1.
func Sign(digestHash []byte, prv *ecdsa.PrivateKey) (sig []byte, err error) {
	if len(digestHash) != DigestLength {
		return nil, fmt.Errorf("hash is required to be exactly %d bytes (%d)", DigestLength, len(digestHash))
	}
	seckey := math.PaddedBigBytes(prv.D, prv.Params().BitSize/8)
	defer zeroBytes(seckey)
	return secp256k1.Sign(digestHash, seckey)
}
```

##### 填充签名数据

把签名数据的这3个值填充到Transaction结构中。

```go
//代码位于 core/types/transaction.go
// WithSignature returns a new transaction with the given signature.
// This signature needs to be in the [R || S || V] format where V is 0 or 1.
func (tx *Transaction) WithSignature(signer Signer, sig []byte) (*Transaction, error) {
	r, s, v, err := signer.SignatureValues(tx, sig)
	if err != nil {
		return nil, err
	}
	cpy := &Transaction{data: tx.data}
	cpy.data.R, cpy.data.S, cpy.data.V = r, s, v
	return cpy, nil
}
```

###### SignatureValues

生成的签名数据是字节数组类型，需要通过signer.SignatureValues()函数转换成3个big.Int类型的数据，然后填充到Transaction结构的R / S / V字段上。

TODO：v 到底怎么回事，一会加27，一会加35

```go
//代码位于 /core/transaction_signing.go
// SignatureValues returns signature values. This signature
// needs to be in the [R || S || V] format where V is 0 or 1.
func (fs FrontierSigner) SignatureValues(tx *Transaction, sig []byte) (r, s, v *big.Int, err error) {
	if len(sig) != crypto.SignatureLength {
		panic(fmt.Sprintf("wrong size for signature: got %d, want %d", len(sig), crypto.SignatureLength))
	}
	r = new(big.Int).SetBytes(sig[:32])
	s = new(big.Int).SetBytes(sig[32:64])
	v = new(big.Int).SetBytes([]byte{sig[64] + 27})
	return r, s, v, nil
}
```



### 提交交易

就看我参考的原文吧。主要是前两点和最近做的东西有点关系。

