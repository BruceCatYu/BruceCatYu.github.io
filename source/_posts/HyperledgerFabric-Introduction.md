---
title: "Hyperldger Fabric简介"
tags: ["中文","入门","概念","已完结","blockchain"]
date: 2022-06-21T15:10:29+08:00
draft: false  
---

# Hyperledger Fabric

* 区块链(分布式账本)
* 联盟链(多中心化)
* 智能合约

![peers](../../../../images/2022/peers.png)  

区块链是由众多参与者(节点)组成的P2P 网络, 每个节点都拥有一个账本的拷贝, 每当有新的交易进来，所有节点的账本都会更新，并且最终保持一致。

## 模型
* **Chaincode**: 链码就是智能合约, 一旦某个事件触发合约中的条款, 代码即自动执行。把约定通过代码的形式，录入区块链中，一旦触发约定时的条件，就会有程序来自动执行，这就是智能合约, 就是业务逻辑  
* **Consensus**: 各个参与者交易信息同步的过程. 共识确保所以参与者都会将同样的信息按同样的顺序更新  
* **Ledger**: 账本是一个有序、防篡改的结构, 是一个个存储着不可修改的数据的区块组成的链表, 每个频道维护一个帐单. 参与者调用链码实现状态转变  
* **Privacy**: 通过频道来隔离帐单, 只有在频道中的参与者才能获取频道内的帐单  
* **Consortium Blockchains**: Hyperledger Fabric属于联盟链: 多个组织通过授权接入，由某些节点参与共识过程  

![peer](../../../../images/2022/peer.png)  
![fabric](../../../../images/2022/fabric.png)  
节点示例  
![organizations](../../../../images/2022/organizations.png)  
组织和节点  
![transaction](../../../../images/2022/transaction.png)  
交易过程  

## References
[基于区块链技术的超级账本 - 从理论到实战](https://blog.csdn.net/i042416/article/details/123365393)  
[Fabric 架构](https://zhuanlan.zhihu.com/p/395644879)  
[Fabric2.0升级智能合约](https://blog.csdn.net/weixin_48814364/article/details/115065282)  
[Fabric2.0设置背书策略](https://blog.csdn.net/weixin_48814364/article/details/115672379)  
[Fabric 2.0 实战 - 设置背书策略](https://learnblockchain.cn/article/617)  
[节点与Channel之间的关系](https://zhuanlan.zhihu.com/p/35349072)  
[官方文档](https://hyperledger-fabric.readthedocs.io/en/release-2.2/whatis.html)  