## external chaincode builder

### 引入原因

当测试一个chaincode时，每个peer都需要使用具体语言（go、java、js）编写完成，同时其他peer也需要explicitly known语言的框架。此外，每个peer都需要单独部署和测试chaincode。当项目部署在容器环境时，例如k8s，chaincode的调试也会变得很困难。

external chaincode builder作为一个服务端调用方法，需要一个权威节点来控制chaincode的构建和部署。虽然ecb没有真正部署在各个peer，但是每个peer依然需要安装ecb的一些信息（hostname,port,TLS configuration）

### chaincode-as-a-service

当创建者创建一个ecb时，external chaincode builder创建一个容器镜像，该镜像会对应每个org的chaincode容器。在这种模式下，builder容器向用户提供了CCAAS的服务，即自动创建出chaincode容器。

### 具体的创建过程

在tutorial中使用了两个terminal来创建和监控这个过程，一个窗口使用monitordocker.sh，另一个窗口使用./network.sh up createChannel来执行具体的创建过程。

使用下面的命令可以执行整个创建过程：

```bash
./network.sh deployCCAAS  -ccn basicts -ccp ../asset-transfer-basic/chaincode-typescript
```

可以将这个过程分为下面三个部分：（顺序无关）

- 打包一个chaincode的容器镜像，并确定了容器的运行方式，包含的contract
- Install, approve, and commit a chaincode definition，chaincode中只包含了连接信息，没有实际代码
- 启动chaincode容器