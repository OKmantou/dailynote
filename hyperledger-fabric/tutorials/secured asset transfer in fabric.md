# Secured asset transfer in fabric

这个tutorial介绍了fabric如何使用state based endorsement, private data, and access control来提供安全的、可验证的、隐私的交易

## Scenario requirements

- asset可以被第一个owner organization发行
- asset的归属权被管理在org层面
- 对asset不可变属性的hash作为asset identifier
- 只有owner知道asset的隐私属性
- buyer需要通过asset的hash来验证asset的隐私属性
- buyer需要验证asset的来源和监管链chain of custody
- buyer和seller都需要为交易背书

## how privacy is maintained

- asset的属性只被隐式地`implicit`保存在owner org的collection中。每个org都有一个collection用于保存这些隐私数据，它们都是隐式地，而不需要显式的定义在chaincode中。
- 在存储隐私信息的hash时会添加salt，以防字典攻击
- 智能合约请求使用 `transient`字段来保存隐私信息，不把隐私信息包括到最终的链上交易信息里
- 只有owner org的peer才可以查询隐私数据

## How transfer is implemented

### creating the asset

部署隐私资产的转移合约时，还需要一个背书策略，以便同一个org的peer在转移asset时不需要其他的peer。asset的创建是一个唯一的交易，该交易只需要链码层面的背书策略。更新或者转移asset的交易接收两方面的治理：state based endorsement policies和 the endorsement policy of private data collection。

- 当智能合约创建asset时，会根据提交的请求获取org的MSP ID，并以kv的形式存储在public chaincode的world state。后续智能合约对asset的访问需要验证是否来自同一org。在其他一些场景下，或许需要更精细的访问控制，而不仅限于同一org的访问
- asset被创建时，智能合约同时会设置一个state based endorsement policies作为asset key。该策略指示了，同一org下的peer必须对asset的操作背书。这防止了其他org使用被恶意修改后的合约来更新或初始化asset。（有点没理解）

### Agreeing to the transfer

transfer的前提：

- asset owner可以在公开关系记录上（public ownership record）改变描述，例如广播asset的售卖信息。智能合约的访问控制保证了，合约只能被owner org提交。state based endorsement保证了，描述只能被owner org的peer修改。

owner和buyer就一个价格达成一致：

- 价格和asset属性被存储在owner和buyer的隐式隐私数据collection。该collection阻止其他org来访问其中的价格和属性。
- 调用private data collection时，价格和属性的hash会自动保存在chain上。这两个hash分别对应两个org就价格和属性达成一致的标记。这用于验证两个org已经达成一致。在price的hash中还会加salt。

### Transferring the asset

在两个org已经达成一致后，就可以调用transfer function来执行转移：

- 对智能合约的访问控制保证了只能由owner org来初始化
- 智能合约中的转移函数要比较buyer和owner的pricate data collection对价格的hash，确保两者是统一价格
- 智能合约还需要验证链上的价格hash，确保两者达成一致
- 如果转移条件满足，从owner的collection中删除asset，并在public state record中更新owner
- 在owner和buyer中删除该asset的隐式private data collection，并且在各自的private data collection中创建一个sales receipts，其中包括交易价格和时间戳
- 更新state based endorsement policy，因为asset owner已经改变
- 