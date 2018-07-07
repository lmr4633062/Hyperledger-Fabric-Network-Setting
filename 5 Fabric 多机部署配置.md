# 5 Fabric 多机部署配置

---

在第 4 章我们介绍了 Fabric 分布式网络单机部署，现在介绍 Fabric 多机部署。我们根据官方 [e2e_cli 例子](https://github.com/hyperledger/fabric/tree/release-1.1/examples/e2e_cli)中的集群方案生成自己的集群，这里把容器分到不同的服务器上，通过网络通信，建立 channel，安装 chaincode。


多机部署需要将 1 个 orderer 节点和 4 个 peer 节点分别部署在 5 台服务器上，5 台服务器可以都按照前 2、3、4 章所述 e2e_cli 的环境构建与测试步骤进行相同的配置。这里我们使用的是虚拟机，固将一台配置好的虚拟机拷贝了 4 份，然后分别对5台虚拟机部署配置。如果使用虚拟机最好将 5 台虚拟机放在一台宿主机内，这样网络方便配置，但是笔者没有那么大的宿主服务器，因此将 4 个虚拟机分别拷贝至其他 4 个 PC 机内。为了方便网络通信，将 5 台宿主机放置于同一网段之下。
## 5.1 配置说明
如上所述，5 台服务器位于同一网段，将会有 4 台作为 peer 节点服务器，1 台作为 orderer 节点服务器，orderer 节点为其他 4 个节点提供排序服务。

各节点参数如下表所示：

名称 | IP | 节点标识|节点 Hostname|Organization 
:-: | :-: | :-: |:-:|:-:
Server 1 | 192.168.119.113 | orderer |orderer.example.com|Orderer
Server 2 | 192.168.119.114 | sp0 |peer0.org1.example.com|Org1
Server 3 | 192.168.119.115 | sp1 |peer1.org1.example.com|Org1
Server 4 | 192.168.119.116 | sp2 |peer0.org2.example.com|Org2
Server 5 | 192.168.119.117 | sp3 |Peer1.org2.example.com|Org2

主要是指修改对应服务器的主机名 Hostname。

## 5.2 生成公私钥、证书、创世区块
公私钥和证书是用于 Server 与 Server 之间的安全通信，另外要创建 channel 并让其它节点加入 channel 就需要首先创建创世区块，公私钥、证书、创世区块都可以通过命令生成，这里官方已经给出了脚本 `generateArtifacts.sh`，在目录 `/opt/gopath/src/github.com/hyperledger/fabric/examples/e2e_cli/` 中可以看到。

在任意一台服务器执行如下命令可以生成公司钥、证书、创世区块：

`bash generateArtifacts.sh mychannel`

将会生成两个目录 `channel-artifacts` 和 `crypto-config`，目录 `channel-artifacts` 包括 `.gitkeep` 、`channel.tx` 、 `genesis.block` 、 `Org1MSPanchors.tx`、 `Org2MSPanchors.tx` 文件，用于 orderer 节点创建 channel。目录 `crypto-config` 包括 `odererOrganizations` 和 `peerOrganizations` 目录，里面包括 orderer 和 peer 的证书、私钥和用于通信加密的 tls 证书等文件。这些文件都是根据配置文件 `configex.yaml` 生成的。

## 5.3 配置多服务器

将 5.2 节中服务器生成的公私钥、证书、创世区块拷贝到其他 4 台服务器里。
即将两个目录 `channel-artifacts` 和 `crypto-config` 拷贝至其他 4 台服务器里相同的目录 `/opt/gopath/src/github.com/hyperledger/fabric/examples/e2e_cli` 下。如果 4 台服务器已经存在相同的 `channel-artifacts` 和 `crypto-config` ，则先删除再拷贝，即保证 5 台服务器具有相同的公私钥、证书、创世区块文件。

## 5.4 配置 peer0.org1.example.com 节点

在Hostname 为 peer0.org1.example.com 的服务器作如下配置：

e2e_cli 中提供了多个 yaml 文件，我们可以基于 docker-compose-cli.yaml 文件创建，具体可执行如下命令：

`cp docker-compose-cli.yaml docker-compose-peer.yaml`

然后修改 `docker-compose-peer.yaml`，去掉 orderer 的配置，只保留一个 peer 和 cli，因为要多级部署，节点与节点之前又是通过主机名通讯，所以需要修改容器中的 host 文件，也就是 extra_hosts 设置，修改后的 peer 配置如下：

```
peer0.org1.example.com:
    container_name: peer0.org1.example.com
    extends:
      file:  base/docker-compose-base.yaml
      service: peer0.org1.example.com
    extra_hosts:
     - "orderer.example.com:192.168.119.113"
```

同样，cli 也需要能够和各个节点通讯，所以 cli 下面也需要添加 extra_hosts 设置，去掉无效的依赖，并且去掉 command 这一行，因为我们是每个 peer 都会有个对应的客户端，也就是 cli ，所以只需要去手动执行一次命令，而无须 command 自动运行。修改后的 cli 配置如下：

```
cli:
    container_name: cli
    image: hyperledger/fabric-tools
    tty: true
    environment:
      - GOPATH=/opt/gopath
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      - CORE_LOGGING_LEVEL=DEBUG
      - CORE_PEER_ID=cli
      - CORE_PEER_ADDRESS=peer0.org1.example.com:7051
      - CORE_PEER_LOCALMSPID=Org1MSP
      - CORE_PEER_TLS_ENABLED=true
      - CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.crt
      - CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.key
      - CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
      - CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/peer
    volumes:
        - /var/run/:/host/var/run/
        - ../chaincode/go/:/opt/gopath/src/github.com/hyperledger/fabric/examples/chaincode/go
        - ./crypto-config:/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/
        - ./scripts:/opt/gopath/src/github.com/hyperledger/fabric/peer/scripts/
        - ./channel-artifacts:/opt/gopath/src/github.com/hyperledger/fabric/peer/channel-artifacts
    depends_on:
      - peer0.org1.example.com
    extra_hosts:
     - "orderer.example.com:192.168.119.113"
     - "peer0.org1.example.com:192.168.119.114"
     - "peer1.org1.example.com:192.168.119.115"
     - "peer0.org2.example.com:192.168.119.116"
     - "peer1.org2.example.com:192.168.119.117"
```

在第 4 章的单机部署模式下，4 个 peer 会映射主机不同的端口，但是在多机部署的时候是不需要映射不同端口的，所以需要修改 `base/docker-compose-base.yaml` 文件，将所有 peer 的端口映射都改为相同的：

```
ports: 
  - 7051:7051 
  - 7052:7052 
  - 7053:7053
```

## 5.5 配置 peer1.org1.excmple.com 节点
与 peer0.org1.example.com 节点 compose 文件配置差不多，不过需要将启动的容器改为 peer1.org1.example.com，并且添加 peer0.org1.example.com 的 IP 映射，对应的 cli 中也改成对 peer1.org1.example.com 的依赖。这里要仔细对照 docker-compose 文件每一行配置代码，尤其是 cli 部分，如果出错会导致后面运行测试出错。

这是修改后的 peer1.org1.example.com 上的配置示例：

```
peer1.org1.example.com:
    container_name: peer1.org1.example.com
    extends:
      file:  base/docker-compose-base.yaml
      service: peer1.org1.example.com
    extra_hosts:
     - "orderer.example.com:192.168.119.113"
     - "peer0.org1.example.com:192.168.119.114"

  cli:
    container_name: cli
    image: hyperledger/fabric-tools
    tty: true
    environment:
      - GOPATH=/opt/gopath
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      - CORE_LOGGING_LEVEL=DEBUG
      - CORE_PEER_ID=cli
      - CORE_PEER_ADDRESS=peer1.org1.example.com:7051
      - CORE_PEER_LOCALMSPID=Org1MSP
      - CORE_PEER_TLS_ENABLED=true
      - CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer1.org1.example.com/tls/server.crt
      - CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer1.org1.example.com/tls/server.key
      - CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer1.org1.example.com/tls/ca.crt
      - CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/peer
    volumes:
        - /var/run/:/host/var/run/
        - ../chaincode/go/:/opt/gopath/src/github.com/hyperledger/fabric/examples/chaincode/go
        - ./crypto-config:/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/
        - ./scripts:/opt/gopath/src/github.com/hyperledger/fabric/peer/scripts/
        - ./channel-artifacts:/opt/gopath/src/github.com/hyperledger/fabric/peer/channel-artifacts
    depends_on:
      - peer1.org1.example.com
    extra_hosts:
     - "orderer.example.com:192.168.119.113"
     - "peer0.org1.example.com:192.168.119.114"
     - "peer1.org1.example.com:192.168.119.115"
     - "peer0.org2.example.com:192.168.119.116"
     - "peer1.org2.example.com:192.168.119.117"
```

## 5.6 配置 peer0.org2.example.com 节点

跟 peer0.org1.example.com 差不多，修改 `docker-compose-peer.yaml` 文件，修改后的 peer 配置如下：

```
peer0.org2.example.com:
    container_name: peer0.org2.example.com
    extends:
      file:  base/docker-compose-base.yaml
      service: peer0.org2.example.com
    extra_hosts:
     - "orderer.example.com:192.168.119.113"

  cli:
    container_name: cli
    image: hyperledger/fabric-tools
    tty: true
    environment:
      - GOPATH=/opt/gopath
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      - CORE_LOGGING_LEVEL=DEBUG
      - CORE_PEER_ID=cli
      - CORE_PEER_ADDRESS=peer0.org2.example.com:7051
      - CORE_PEER_LOCALMSPID=Org2MSP
      - CORE_PEER_TLS_ENABLED=true
      - CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/server.crt
      - CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/server.key
      - CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
      - CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/peer
    volumes:
        - /var/run/:/host/var/run/
        - ../chaincode/go/:/opt/gopath/src/github.com/hyperledger/fabric/examples/chaincode/go
        - ./crypto-config:/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/
        - ./scripts:/opt/gopath/src/github.com/hyperledger/fabric/peer/scripts/
        - ./channel-artifacts:/opt/gopath/src/github.com/hyperledger/fabric/peer/channel-artifacts
    depends_on:
      - peer0.org2.example.com
    extra_hosts:
     - "orderer.example.com:192.168.119.113"
     - "peer0.org1.example.com:192.168.119.114"
     - "peer1.org1.example.com:192.168.119.115"
     - "peer0.org2.example.com:192.168.119.116"
     - "peer1.org2.example.com:192.168.119.117"
```

## 5.7 配置 peer1.org2.example.com 节点

跟上述几个节点类似，修改后的 peer 配置如下：

```
peer1.org2.example.com:
    container_name: peer1.org2.example.com
    extends:
      file:  base/docker-compose-base.yaml
      service: peer1.org2.example.com
    extra_hosts:
     - "orderer.example.com:192.168.119.113"
     - "peer0.org2.example.com:192.168.119.116"

  cli:
    container_name: cli
    image: hyperledger/fabric-tools
    tty: true
    environment:
      - GOPATH=/opt/gopath
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      - CORE_LOGGING_LEVEL=DEBUG
      - CORE_PEER_ID=cli
      - CORE_PEER_ADDRESS=peer1.org2.example.com:7051
      - CORE_PEER_LOCALMSPID=Org2MSP
      - CORE_PEER_TLS_ENABLED=true
      - CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer1.org2.example.com/tls/server.crt
      - CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer1.org2.example.com/tls/server.key
      - CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer1.org2.example.com/tls/ca.crt
      - CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/peer
    volumes:
        - /var/run/:/host/var/run/
        - ../chaincode/go/:/opt/gopath/src/github.com/hyperledger/fabric/examples/chaincode/go
        - ./crypto-config:/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/
        - ./scripts:/opt/gopath/src/github.com/hyperledger/fabric/peer/scripts/
        - ./channel-artifacts:/opt/gopath/src/github.com/hyperledger/fabric/peer/channel-artifacts
    depends_on:
      - peer1.org2.example.com
    extra_hosts:
     - "orderer.example.com:192.168.119.113"
     - "peer0.org1.example.com:192.168.119.114"
     - "peer1.org1.example.com:192.168.119.115"
     - "peer0.org2.example.com:192.168.119.116"
     - "peer1.org2.example.com:192.168.119.117"
```

## 5.8 配置 orderer 节点

与创建 peer 的配置文件类似，首先复制一个 yaml 文件并对其进行修改：

`cp docker-compose-cli.yaml docker-compose-orderer.yaml`

orderer 服务器上只需要保留 order 设置，其他 peer 和 cli 设置都可以删除。orderer 可以不设置 extra_hosts。

以下是 orderer-compose 的配置示例：

```
orderer.example.com:
    extends:
      file:   base/docker-compose-base.yaml
      service: orderer.example.com
    container_name: orderer.example.com
```
