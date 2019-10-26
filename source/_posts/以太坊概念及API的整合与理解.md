---
title: 以太坊概念及API的整合与理解
date: 2019-10-22 17:21:16
categories:
- 区块链
tags:
- 以太坊
- API
- Javascript
---

## Ethereum Readme

### Executables

go-ethereum 项目的`cmd`文件夹下，有很多包装器/可执行文件

| 命令        | 描述                                                         |
| ----------- | ------------------------------------------------------------ |
| geth        | 主要的以太坊命令行接口（CLI）客户端，可运行全节点等类型节点。它可以被其他进程用作网关，通过暴露在HTTP，WebSocket和 IPC 传输之上的 JSON RPC 端点进入以太坊网络。 |
| abigen      |                                                              |
| bootnode    | 以太坊客户端实现的精简版，只加入网络的节点发现协议，不运行其他的更高层应用层协议。可作为轻量级启动节点，协助私有网络中的节点发现。 |
| evm         | 开发实用工具                                                 |
| gethrpctest | 开发实用工具                                                 |
| rlpdump     | 开发实用工具                                                 |
| puppeth     | CLI向导程序，帮助创建新的以太坊网络                          |

### Running `geth`

命令行标记：[Command Line Options](<https://github.com/ethereum/go-ethereum/wiki/Command-Line-Options>)

`geth [options] command [command options] [arguments...]`

```
ETHEREUM OPTIONS:
LIGHT CLIENT OPTIONS:
DEVELOPER CHAIN OPTIONS:
ETHASH OPTIONS:
TRANSACTION POOL OPTIONS:
PERFORMANCE TUNING OPTIONS:
ACCOUNT OPTIONS:
API AND CONSOLE OPTIONS:
NETWORKING OPTIONS:
MINER OPTIONS:
GAS PRICE ORACLE OPTIONS:
VIRTUAL MACHINE OPTIONS:
LOGGING AND DEBUGGING OPTIONS:
METRICS AND STATS OPTIONS:
WHISPER (EXPERIMENTAL) OPTIONS:
DEPRECATED OPTIONS:
MISC OPTIONS:
```

`geth console`命令 ：1）以 fast sync 模式开启 `geth`；2）启动 `geth`内建的  [JavaScript console](<https://github.com/ethereum/go-ethereum/wiki/JavaScript-Console>) （通过 `console` 子命令），调用所有的官方的 web3 methods [JavaScript API](<https://github.com/ethereum/wiki/wiki/JavaScript-API>) 和 `geth` 本身的 [management APIs](<https://github.com/ethereum/go-ethereum/wiki/Management-APIs>) 。`console` 是可选项，不写时会 `attach` 到一个正在运行的 `geth` 实例（默认是生产节点），即 `geth attach`。

_____________________________________________
### [JavaScript console](<https://github.com/ethereum/go-ethereum/wiki/JavaScript-Console>)  

- 以太坊提供了一个  **javascript runtime environment** (JSRE)，可用在交互式（console）或 非交互式（script）模式。 Javascript console 暴露了所有的 web3 Javascript Dapp API( [JavaScript API](<https://github.com/ethereum/wiki/wiki/JavaScript-API#web3ethcall>) )和 admin API ( [JavaScript Console](<https://github.com/ethereum/go-ethereum/wiki/JavaScript-Console#javascript-console-api>) )

- 交互式应用，the JSRE REPL Console（**Read, Evaluate & Print Loop**）：

  - `geth console` ： 开启 geth 节点，并打开控制台

  -  `geth attach`：打开正在运行的 geth 实例的控制台 
- geth 节点在非默认的 ipc 端点下运行，则可以通过 rpc 接口连接
  
- note：默认情况下 geth 开启时不启动 http 和 ws 服务，没有提供完整接口
      - `$ geth attach ipc:/some/custom/path `
  	-  `$ geth attach http://191.168.1.1:8545`
    	 	 -  `$ geth attach ws://191.168.1.1:8546 `
  
  - 记录日志信息，不在控制台上打印内容： `$ geth --verbosity 5 console 2>> /tmp/eth.log`
  - 加载自定义 JavaScript 文件（加载常用函数，设置web3 合约对象）： `geth --preload "/my/scripts/folder/utils.js,/my/scripts/folder/contracts.js" console`
  
- 交互式应用：JSRE script mode

  - 还可以向JavaScript解释器执行文件。某种复杂形式：`$ geth --jspath "/tmp" --exec 'loadScript("checkbalances.js")' attach http://123.123.123.123:8545`

### [JavaScript API](https://github.com/ethereum/wiki/wiki/JavaScript-API)

要使应用程序在 Ethereum上 工作，可以使用 web3.js 库提供的 web3 对象。在底层，它通过RPC调用与本地节点通信。web3.js 处理任何暴露 RPC 层的 Ethereum 节点。web3 包含 `eth`对象（`web3.eth` 与以太坊区块链的交互），`ssh`对象（`web3.ssh` 与Whisper 的交互），`net`  `db` `ssh`。

- Adding web3
- Using callbacks：本协议是与本地的 RPC 节点工作，它的所有功能默认使用同步的 HTTP 请求。
- Batch requests
- A note on big numbers in web3.js
  - 浮点数建议小数部分是 20位，所以 `balance` 一般都建议用 `Wei`，在给用户展示的时候再转换为 `ether`。

###  [management APIs](<https://github.com/ethereum/go-ethereum/wiki/Management-APIs>) 

#### Enabling the management APIs

要通过 Geth RPC 端点提供这些 api，需要用命令 `--${interface}api` 命令行参数， `--${interface}`可以是 `rpc,ws,ipc`。

`geth --ipcapi admin,eth,miner --rpcapi eth,web3 --rpc`

- 在 IPC 接口上启用 admin、官方 DAp p和 miner API；
- 通过 HTTP 接口启用 官方的DApp 和 web3 API；
- 必须使用 `--rpc` 标志显式地启用 HTTP RPC 接口

#### List of management APIs

- `admin`: Geth node management
- `debug`: Geth node debugging
- `miner`: Miner and [DAG](https://github.com/ethereum/wiki/wiki/Ethash-DAG) management
- `personal`: Account management
- `txpool`: Transaction pool inspection

见 managementAPI 图

![managementAPI.JPG](https://i.loli.net/2019/10/23/UJheoOLq7MBvzxI.jpg)

________



#### Configuration

- `geth` 后面的多个标记可以写在配置文件中，如  `$ geth --config /path/to/your_config.toml`。

- 查看配置文件的内容，可使用命令： `$ geth --your-favourite-flags dumpconfig` 导出已存的配置。
- 只在 geth v1.6.0 以上的版本使用。

#### Docker quick start

`--rpcaddr 0.0.0.0`：如果想从其他容器或者主机访问 RPC。默认情况下，`geth` 绑定到本地的接口和 RPC 端点，外部无法访问。

#### Programmatically interfacing（连接） `geth` nodes

通过自己的程序去连接 `geth` 和以太坊网络，而不是手动地用 `console` 。`geth` 内置了对基于JSON-RPC 的 api 的支持：标准的 APIs（[JSON RPC](<https://github.com/ethereum/wiki/wiki/JSON-RPC>)）和 geth 特定的 APIs（[Management APIs](<https://github.com/ethereum/go-ethereum/wiki/Management-APIs>)）。可以通过 HTTP，WebSockets 和 IPC（基于 UNIX 平台的 UNIX 套接字，windows 上称作 pipes） 去暴露。

默认情况下，IPC 接口打开，暴露 `geth`支持的所有APIs，HTTP 和 WS 接口需要手动打开，只暴露一部分 APIs，可通过配置关闭或者打开。

使用自己的编程环境的功能(库、工具等)通过 HTTP、WS 或 IPC 连接到 geth 节点。需要在所有传输上使用 JSON-RPC。可以对多个请求重用相同的连接!

HTTP based JSON-RPC API options：

- `--rpc` Enable the HTTP-RPC server
- `--rpcaddr` HTTP-RPC server listening interface (default: `localhost`)
- `--rpcport` HTTP-RPC server listening port (default: `8545`)
- `--rpcapi` API's offered over the HTTP-RPC interface (default: `eth,net,web3`)
- `--rpccorsdomain` Comma separated list of domains from which to accept cross origin requests (browser enforced)
  - 参见：[跨域资源共享 CORS 详解](<http://www.ruanyifeng.com/blog/2016/04/cors.html>)

#### [JSON RPC](<https://github.com/ethereum/wiki/wiki/JSON-RPC>)