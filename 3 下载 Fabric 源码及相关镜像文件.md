# 3 下载 Fabric 源码及相关镜像文件

---

## 3.1 下载 Fabric 源码
Fabric 多机分布式网络搭建过程中需要使用 Fabric 源码中的工具，工具编译需要用 go 语言环境，因此需要把源码目录放到 `$GOPATH` 下。通过 2.4 中 go 的安装配置， `$GOPATH` 已经设置为 /opt/gopath。

源码安装前需要构建本地 git 环境：

`sudo yum install git`

然后下载源码：

`git clone github.com/hyperledger/fabric`

下载完成后进入源码目录：

`cd /opt/gopath/src/github.com/hyperledger/fabric/`

如果不想使用最新版，可以切到之前的版本，如切到 v1.0.0：

`git checkout -b v1.0.0`

然后在下载目录 /opt/gopath/src/github.com/hyperledger/fabric/ 下看到从 [Github源码](/opt/gopath/src/github.com/hyperledger/fabric/) 下载的各文件。

## 3.2 下载相关镜像文件
相关镜像文件的下载可以通过执行脚本自定下载和手动下载 2 种方法，任选一种即可。
### 3.2.1 脚本自动下载
进入 fabric/examples/e2e_cli 目录下执行下载脚本 download-dockerimages.sh，即可下载所需要的镜像：

`cd examples/e2e_cli`

`./download-dockerimages.sh`
### 3.2.2 手动下载
这种方法可以自己控制镜像版本号，升级方便。

所有镜像均可在[Docker Hub 官方网站](https://hub.docker.com/) 通过搜索  [hyperledger](https://hub.docker.com/r/hyperledger/) 搜到。

Fabric 需要用到的镜像主要有以下几种：

hyperledger/fabric-tools
hyperledger/fabric-orderer
hyperledger/fabric-peer
hyperledger/fabric-couchdb
hyperledger/fabric-kafka
hyperledger/fabric-ca
hyperledger/fabric-ccenv
hyperledger/fabric-baseimage


以 hyperledger/fabric-tools 为例：首先点击 DETAILS  进入详情页，然后点击 Tags 查看不同版本列表，查看当前 fabric-tools 最新版本号，根据我们所使用的操作系统情况，选择对应的 x86_64 版本号，如本文使用 v1.1.0 版本，因此选择 x86_64-1.1.0版本 ，故最终执行的 docker 下载命令如下：

`docker pull hyperledger/fabric-tools:x86_64-1.1.0`

根据这种方法，可以将上述必要的镜像由 docker 服务全部下载至本地，并最终使用 docker-compose 来启动对应的镜像服务。

为了方便 docker-compose 的配置，我们将所有的镜像 tag 都改为 latest ，执行如下格式的命令：

`docker tag IMAGEID(镜像id) REPOSITORY:TAG（仓库：标签）`

如：

`docker tag 0403fd1c72c7 docker.io/hyperledger/fabric-tools:latest`

全部修改完成后，执行命令 `docker images` 查看下载的镜像及修改的 tag 是否成功。




