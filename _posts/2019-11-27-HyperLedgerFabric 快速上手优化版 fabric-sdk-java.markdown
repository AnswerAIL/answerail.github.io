---
layout: post
title: HyperLedgerFabric 快速上手优化版 fabric-sdk-java
tags: [区块链, 教程类]
image: '/images/posts/1.jpg'
---

### 1. 前言
> &ensp;&ensp;&ensp;由于 fabric-sdk-java 存在普遍的上手难问题， 官方 Java 版 sdk 单元测试 demo 比较难理解， 对于想要通过使用 java-sdk 入门 fabric 的朋友来说， 这个是一个比较大的阻碍。 鉴于此， 本博文重新整理了下 Java 版 sdk 的使用， 在官方源码的基础上进行重新封装一层， 目的就是消除这道入门障碍， 让大家都可以使用 java-sdk 来拥抱 fabric。
> > ***[fabric-sdk-server项目地址](https://github.com/AnswerAIL/fabric-sdk-server)*** 当前项目版本只适合大家入门fabric， 对于支持 tls 以及 ca 等其他功能的适配， 由于个人时间及工作的原因， 无法在现有版本上进行支持，后续有机会可能会在新版本中发布。

&nbsp;

### 2. 前置条件
 - JDK 1.8， [下载地址](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)
 - 安装 redis 服务， [下载地址](http://download.redis.io/)
 - 已成功搭建 fabric 区块链网络， 区块链网络搭建可参考《 [hyperledger fabric 1.1区块链网络环境部署及cli实操](https://blog.csdn.net/u010979642/article/details/88739822)》
 
&nbsp;

### 3. 区块链网络修改
```bash
############################################################
# 关闭每次重启fabric网络时重新生成通道及加密配置文件信息(建议)
############################################################
# 首次启动 network_setup.sh 脚本后, 修改以下代码, 重启 Fabric 服务
vim network_setup.sh

# 删除 network_setup.sh 脚本中的以下代码
#    function networkUp   -> source generateArtifacts.sh $CH_NAME   # 删除 else 整个片段的代码
#    function networkDown -> rm -rf channel-artifacts/*.block channel-artifacts/*.tx crypto-config


############################################################
# 注释掉tls配置(必须)
############################################################
cd fabric/examples/e2e_cli

vim base/peer-base.yaml
    CORE_PEER_TLS_ENABLED=true 
    #   改为
    CORE_PEER_TLS_ENABLED=false


vim base/docker-compose-base.yaml
    ORDERER_GENERAL_TLS_ENABLED=true
    #   改为
    ORDERER_GENERAL_TLS_ENABLED=false

vim docker-compose-cli.yaml
    CORE_PEER_TLS_ENABLED=true
    #   改为
    CORE_PEER_TLS_ENABLED=false


############################################################
# 注释掉 script.sh 脚本中原有通过 cli 命令行操作区块链网络的相关代码
############################################################
cd fabric/examples/e2e_cli

vim scripts/script.sh        
# 注释块起始位置(包含以下部分)  
    ## Create channel
    echo "Creating channel..."
    createChannel
# 注释块截至位置(包含以下部分)
    #Query on chaincode on Peer3/Org2, check if the result is 90
    echo "Querying chaincode on org2/peer3..."
    chaincodeQuery 3 90   


############################################################
# 启动区块链网络, 先关闭再启动, 或 直接 restart
############################################################
bash network_setup.sh down
bash network_setup.sh up                
```

&nbsp;

### 4. SDK 操作步骤
 1. 拉取 github 仓库指定版本代码到本地 - https://github.com/AnswerAIL/fabric-sdk-server.git
    - [fabric-sdk-server release 1.0](https://github.com/AnswerAIL/fabric-sdk-server/tree/release-1.0)
    - [fabric-sdk-server release 1.1](https://github.com/AnswerAIL/fabric-sdk-server/tree/release-1.1)

 2. 配置 [com.hyperledger.fabric.sdk.common.Constants](https://github.com/AnswerAIL/fabric-sdk-server/blob/master/src/main/java/com/hyperledger/fabric/sdk/common/Constants.java) 类中 redis 服务的信息
 
 3. 配置 [com.hyperledger.fabric.sdk.common.Config](https://github.com/AnswerAIL/fabric-sdk-server/blob/master/src/test/java/com/hyperledger/fabric/sdk/common/Config.java) 类中 区块链网络的节点信息

 4. 替换掉 answer-fabric-sdk\src\test\resources 下的配置文件， 注意是  XXX/`test`/resources 
    - chaincodes/*   &ensp;&ensp;  `智能合约， 如果你测的是官方转账功能， 可不用替换`
    - channel-artifacts/*   &ensp;&ensp; `通道配置信息及创世纪块等， 可直接删掉， 将工具生成的目录拷贝过来`
    - config.properties   &ensp;&ensp; `官方sdk配置文件， 可不必替换`
    - crypto-config/*   &ensp;&ensp; `证书签名密钥等文件， 可直接删掉， 将工具生成的目录拷贝过来`
    - policy/*   &ensp;&ensp; `背书策略文件， 可不用替换`

> 说明： 考虑到方便大家上手， channel-artifacts 和 crypto-config 目录的文件采用完成替换模式， 即： 删除掉该项目这两个文件夹下的文件， 然后直接将你生成好的这两个目录原封不动的拷贝过来即可。 项目下的智能合约代码为官方转账功能代码， 即：chaincode_example02.go
> > ***最便捷的方式：*** 第4步只需替换 channel-artifacts 和 crypto-config 两个目录的文件即可。

 5. 使用以下测试用例即可通过sdk来操作区块链网络
    - APITest.java   &ensp;&ensp;   `模拟 全流程 操作区块链网络 `
    - BlockChainTest.java   &ensp;&ensp;  `查询区块 | 账本信息`
    - InvokeTest.java   &ensp;&ensp;  `模拟转账操作`
    - QueryTest.java   &ensp;&ensp;  `查询智能合约`
    - UpgradeTest.java   &ensp;&ensp;  `升级智能合约`
    - JoinPeerTest.java   &ensp;&ensp;  `加入新节点`

> 建议按如下顺序执行测试用例： APITest -> JoinPeerTest -> QueryTest -> InvokeTest -> QueryTest -> UpgradeTest -> QueryTest 

&nbsp;
 
### 5. 相关网址
 - [x] [fabric-sdk-java 源码](http://github.com/hyperledger/fabric-sdk-java)
 - [x] [接入Fabric E2E案例流程说明](https://github.com/AnswerAIL/fabric-sdk-server/blob/master/%E6%8E%A5%E5%85%A5Fabric%20E2E%E6%A1%88%E4%BE%8B%E6%B5%81%E7%A8%8B%E8%AF%B4%E6%98%8E.md)
