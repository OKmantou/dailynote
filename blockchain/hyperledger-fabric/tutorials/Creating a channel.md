# Creating a channel

为了简化channel的创建过程，这里使用 `configtxgen`工具创建创世区块和  `osnadmin CLI`将ordering service加入到channel中， 不用首先创建一个ordering service管理的“system channel”。

**这样做的好处**

- increased privacy：之前的所有order node都需要加入到system channel中，任何channel的创建都会被order node看到，即使这个node并没有加入到该channel中。
- 可扩展性：当system channel上有大量的ordering nodes时，启动一个node会导致所有的order同步所有的channel ledger。同时大量的nodes增加了nodes间的心跳同步时延。

