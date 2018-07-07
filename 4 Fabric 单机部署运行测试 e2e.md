# 4 Fabric 单机部署运行测试 e2e

---

## 4.1 运行 e2e_cli 项目
进入到/opt/gopath/src/github.com/hyperledger/fabric/examples/e2e_cli目录下，看到如 [Github项目](https://github.com/hyperledger/fabric/tree/release-1.1/examples/e2e_cli) 所示文件。

使用如下命令进入 e2e_cli 目录：

`cd /opt/gopath/src/github.com/hyperledger/fabric/examples/e2e_cli`

network_setup.sh 是一件测试脚本，该脚本启动 5 个 docker 容器，其中 4 个容器运行 peer 节点和 1 个容器运行 orderer 节点，它组成一个 Fabric 集群。另外还有一个 cli 容器用于执行创建 channel、加入 channel、安装和执行 chaincode 等操作。测试用的 chaincode 定义了两个变量，在实例化的时候会给这两个变量赋予了初始值，并通过 invoke 操作可以使两个变量的值发生变化。

执行脚本：

`bash network_setup.sh up`

看到以下信息时说明测试通过

```
Query Result: 90
2017-05-16 17:08:15.158 UTC [main] main -> INFO 008 Exiting.....
======== Query successful on peer1.org2 on channel 'mychannel' ========

================= All GOOD, BYFN execution completed ==================
 _____   _   _   ____
| ____| | \ | | |  _ \
|  _|   |  \| | | | | |
| |___  | |\  | | |_| |
|_____| |_| \_| |____/
```

这个命令可以在本机启动 4+1 的 Fabric 网络并且进行测试，跑 Example02 这个 ChainCode。我们可以看到每一步的操作，最后确认单机没有问题。确认我们的镜像和脚本都是正常的，我们就可以关闭 Fabric 网络。关闭 Fabric 命令：

`bash network_setup.sh down`

如果是在虚拟机中部署的，上述测试成功后可以创建快照，以便以后遇到问题可以恢复快照重来。

## 4.2 运行问题解决
### 问题 1 
该 Fabric 网络集群测试环境在 Linux 内核低版本上可能会出现问题，如执行 chaincode 初始化的时候报错，导致集群单机无法启动，这是旧版内核的bug。

解决方案，使用最新版稳定版的 docker， 在第 2 章我们如果下载的是最新版本的 docker 就无须升级了（笔者写文档时的所下最新稳定版为：18.04.0-ce-rc2）；将 Linux 内核升级到最新版（笔者使用的内核版本是 4.16.0-1.el7）。

查看内核版本命令：

`uname -a`
### 问题 2
遇到如下问题导致 chaincode 初始化失败
```
Error: Error endorsing chaincode: rpc error: code = Unknown desc = Error starting container: API error (404): {"message":"network e2ecli_default not found"}
```
解决方案：重命名文件夹 e2e_cli，改为 e2ecli 后再执行 `bash network_setup.sh up` 测试成功。
