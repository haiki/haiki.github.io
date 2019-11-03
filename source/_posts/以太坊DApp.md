---
title: 以太坊DApp
date: 2019-10-26 18:55:09
categories:
- 区块链
tags:
- ethereum
- DApp
---

## [Web开发者视角下解释以太坊](<https://medium.com/@mvmurthy/ethereum-for-web-developers-890be23d1d0c>)

### web App 在客户端-服务器架构下的工作原理

如图 webApp

![webApp.png](https://i.loli.net/2019/10/26/GaObWcxqQBFyIDE.png)

- web应用程序托管在主机提供商上
- 客户端（可以是浏览器、使用服务的另一个 api 等）向服务器发出请求
- 当客户机向服务器发出请求时，服务器执行它的操作，与数据库和/或缓存通信，读取/写入/更新数据库并为客户机服务
- 比如闲鱼，外卖，淘宝？提供了一个安全可信的中间商平台，商家和买家其实都要为其服务支付费用

如果：每个人都能公开且安全地访问数据库，不必依赖这个 webapp 所有者来获取数据，可以节省佣金，也可以访问你的所有数据  ——  分布式 Dapps

### 以太坊 Dapp 的架构

如图：ethereumDapp

![ethereumDapp.png](https://i.loli.net/2019/10/26/r8UEeV4cWCXZvTw.png)

- 每个客户机(浏览器)都与自己的应用程序实例进行通信
- 在使用应用程序之前，必须下载整个区块链

### 以太坊是什么？

1. 数据库（Database）：存储数据；交易，区块
2. 代码（code）：具体应用的逻辑，买、卖、取消，退款等，需要编写应用层代码，即合约。将合约编译成以太坊字节码，将字节码部署在区块链上。

因此，以太坊存储你的数据，存储代码，并在 EVM 中执行代码。

web3.js 是一个 JavaScript 库，用来连接以太坊节点。将这个库包含（include）进 js 框架中，即可开始构建。

