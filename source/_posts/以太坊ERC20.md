---
title: 以太坊ERC20
date: 2019-12-23 10:13:29
categories:
- 区块链
tags:
- 以太坊
- solidity
- 智能合约
- token
---

- 译文：[2017-理解ERC-20 token合约-以太坊爱好者](https://mp.weixin.qq.com/s?__biz=MzIwODA3NDI5MA==&mid=2652525287&idx=1&sn=4e8409169abb97a7ba36471cece34ad3&chksm=8ce651babb91d8ac9c6a69f25344981a11d29e799940dbce953b0b43cdb6ea65f64ec508b842&scene=21#wechat_redirect) 
  - 原文：[2017-Understanding ERC-20 token contracts-Jim McDonald](https://www.wealdtech.com/articles/understanding-erc20-token-contracts/)

从这篇文章中可知，ERC20 就是一个记录 <地址，资产> 的键值存储数据库。所有的 存 token、取 token 操作相当于在记录里面的增/删/改一条记录。

- [2018-教你如何一步步创建ERC20代币-Pony小马](https://www.jianshu.com/p/4002a5212885)

这篇文章实践为例，构建简单的 ERC20 代币，介绍了ERC20的标准接口并做了实现和部署。

- 译文：[2017-代币支付的以太坊智能服务-以太坊爱好者](https://mp.weixin.qq.com/s/foM1QWvsqGTdHxHTmjczsw)
  - 原文：[2017-Ethereum smart service payment with tokens-Jim McDonald](https://www.wealdtech.com/articles/ethereum-smart-service-payment-with-tokens/)

这篇文章主要以伪代码的形式，说明了外部服务如何使用 token 作为一种支付方式。以图文并茂的方式告诉我们几个函数的适用场景。包括 ：`approve()`，`transfer()`，`transferFrom()`，`approveAndCall()`，`receiveApproval()`，`transferAndCall()`，`tokenFallback()`。

- [The 2 ways to transfer ERC20 Tokens in Solidity](https://dev.to/jklepatch/the-2-ways-to-transfer-erc20-tokens-in-solidity-3h40)
  - transfer() (simple transfers)

    - Import the ERC20 token
      - `import  "github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/ERC20.sol";`
    - Import the address of the token
  - Call transfer()
  
- transferFrom() (delegated transfer)
  
    - Import the ERC20 token
    - Import the address of the token
    - Call approve()
  - Call transferFrom()
  
    