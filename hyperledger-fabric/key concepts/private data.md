##  private data

### private data

在一个channel中，不是所有信息都希望所有org的所有peer看到，需要有一部分隐私信息。为了满足这个需求，可以重新建立一条channel。

但是这样会带来新的问题（additional administrative overhead），例如这条拆分的包含隐私信息的channel需要与其他channel保持内容一致性，会增加channel之间的依赖。因此需要一种机制既要保证channel中部分信息的隐私（对部分org和peer可见），又要满足公开信息和隐私信息的正确性，减少信息的不一致。

fabric提供了一种private data collection，是一个若干orgs的子集，该子集的org可以endorse、commit、query。

private data collections可以显式地被定义在chaincode中，并且每个chaincode有一个隐式地隐私数据namespace用于保护org指定地隐私数据

### private data collection

> private data collection的使用场景：
>
> 1. 交易在一部分orgs中传播，但是只有更小一部分orgs有权访问交易数据。
> 2. 保证交易信息对ordering service nodes保密性，交易只在peers间传播，而不是在blocks中（无需经过排序）

一个collection由两个部分组成：

- 真实的隐私数据

  使用gossip协议在peer间广播。隐私数据被存储在已经被org授权的peer节点上。因为使用gossip协议，所以还需要设置anchor peers

  > 隐私数据是怎么传递的，怎么保证隐私数据只在授权的peers间传播

- 数据的哈希

### use case

