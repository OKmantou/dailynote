## Using private data in fabric

### Asset transfer private data sample use case

三个collections：`assetCollection`，`Org1MSPPrivateCollection`，`Org2MSPPrivateCollection`

Org1的peer创建了一个新的asset，作为该asset的owner，并且在assetCollection中存储了有关asset owner的identity。assetCollection是创建在channel层面的。

该asset同时会由owner提供一个appraised value，appraised value 用于参与者之间转移资产，该value被存储在owner org的collection中，在本例子中是Org1MSPPrivateCollection。

购买者（org2中的peer）需要调用智能合约`AgreeToTfansfer`来完成交易，并且同意以appraisal value成交。成交值被存储在Org2MSPPrivateCollection。

现在asset owner可以使用智能合约函数`TransferAsset`转移asset，该函数会验证owner和buyer已经就appraised value达成一致。

### Build a collection definition JSON file

使用private data collection时，要在channel中定义collection definition file。被定义在private data collection中的数据只能在某一组织内的成员中传递。

但是在同一个channel中的所有org都应该部署该config，即使org不属于任何一个collection

 一个collection definition包括下面的字段：

- policy：被允许保存隐私数据的peers
- requiredPeerCount：传递隐私数据的最少peer数目
- maxPeerCount：为了达到冗余目的
- blockToLive：为存储在数据库中的隐私数据设置一个存活时间，单位是block，出块的个数
- memberOnlyRead：
- memberOnlyWrite：
- endorsementPolicy：覆盖channel层面的policy

### Read and write private data using chaincode APIs

这部分介绍如何在channel上私有化数据。在上面的例子中，私有数据被分为三个数据定义：`Asset`，`AssetPrivateDetails`和 `AssetPrivateDetails`，其中后两个是分别在org1和org2中定义

**Reading collection data**

使用`GetPrivateData()`函数来查询，需要两个参数：`collection name`和 `data key`

**Writing private data**

`PutPrivateData()`

### 3. Deploy the private data smart contract to the channel

collection作为智能合约的一部分部署到channel上。智能合约上包括了chaincode层面的背书策略，所以org不用再需要其他org 的背书就可以创建asset

部署完智能合约后，还需要让org中的所有节点接受collection

### 4. Register identities 

### 5. Create an asset in private data

### 6. Query the private data as an un/authorized peer

### 7. Transfer the Asset

