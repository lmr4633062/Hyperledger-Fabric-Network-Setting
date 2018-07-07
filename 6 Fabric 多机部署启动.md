# 6 Fabric 多机部署启动


---

启动 Fabric 多节点集群

## 6.1 启动 orderer 节点服务

完成第 5 章操作后，各节点的 compose 配置文件及证书验证目录都已经准备完成，可以开始尝试启动多机 Fabric 集群。

首先启动 orderer 节点，此操作在 orderer.example.com 服务器上完成，即前文指定的 IP 地址 192.168.119.113 的服务器，执行如下命令进入启动 docker 进程：

`docker-compose -f docker-compose-orderer.yaml up -d`

运行完毕后执行 `docker ps` 看到运行了一个名字为 orderer.example.com 的节点。如下所示：

```
CONTAINER ID        IMAGE                        COMMAND             CREATED             STATUS              PORTS                    NAMES
994164102c1d        hyperledger/fabric-orderer   "orderer"           About an hour ago   Up 3 seconds        0.0.0.0:7050->7050/tcp   orderer.example.com
```
## 6.2 启动 peer 节点服务

切换到 peer0.org1.example.com 服务器，即 IP 地址为 192.168.119.114 的服务器，启动本服务器的 peer 节点和 cli，执行如下命令：

`docker-compose -f docker-compose-peer.yaml up -d`

运行完毕后执行命令 `docker ps` 应该可以看到2个正在运行的容器 cli 和 peer0.org1.example.com。

同理执行命令 `docker-compose -f docker-compose-peer.yaml up -d`，在另外 3 台服务器启动 peer 节点容器。

## 6.3 创建 channel 和安装chaincode

切换到 peer0.org1.example.com 服务器上，使用该服务器上的 cli 来运行创建 Channel 和安装 ChainCode 的操作。首先需要进入 cli 容器，执行如下命令： 

`docker exec -it cli bash `

进入容器后可以看到命令提示变为如下所示： 

```
root@dd815a900955:/opt/gopath/src/github.com/hyperledger/fabric/peer#
```

说明现在已经以 root 的身份进入到 cli 容器内部。官方已经提供了完整的创建 Channel 和测试 ChainCode 的脚本，并且已经映射到 cli 容器内部，所以只需要在 cli 内运行如下命令： 

`./scripts/script.sh mychannel`

该脚本会一步一步的完成创建通道，将其他节点加入通道，更新锚节点，安装 ChainCode，初始化账户，查询，转账，再次查询等链码的各个操作都可以自动化实现。直到最后，系统提示如下信息：

```

============== All GOOD, End-2-End execution completed ================
 _____   _   _   ____
| ____| | \ | | |  _ \
|  _|   |  \| | | | | |
| |___  | |\  | | |_| |
|_____| |_| \_| |____/
```

说明 4+1 的 Fabric 多机部署成功了。现在是在 peer0.org1.example.com 的 cli 容器内，切换到 peer0.org2.example.com 服务器，运行 `docker ps` 命令，可以看到 peer0.org1.example.com 执行 `./scripts/script.sh mychannel` 前本来是 2 个容器的，现在已经变成了 3 个容器，因为 ChainCode 会创建一个容器。

这里使用官方脚本自动化执行的各个操作，如果不想使用脚本也可以根据[官网](http://hyperledger-fabric.readthedocs.io/en/latest/build_network.html)的指南，通过手动方式一台服务器一台服务器地配置，分别执行加入通道，更新锚节点，创建 ChainCode，初始化账户，查询，转账，再次查询等链码的各个操作。

参考资料：
1 [Hyperledger Fabric 1.0 从零开始](http://www.cnblogs.com/aberic/p/7527831.html)

2 [A Blockchain Platform for the Enterprise](http://hyperledger-fabric.readthedocs.io/en/release-1.2/whatis.html)

3 [Setting up a Blockchain Business Network With Hyperledger Fabric & Composer Running in Multiple Physical Machine](https://www.skcript.com/svr/setting-up-a-blockchain-business-network-with-hyperledger-fabric-and-composer-running-in-multiple-physical-machine/)
