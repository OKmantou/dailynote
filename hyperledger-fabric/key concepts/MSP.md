# MSP

## identity

身份确定了一个角色拥有的对资源的操作权限和对信息的访问权限。在fabric中，数字身份包含了描述这个角色的若干属性。一个可以验证的身份必须被可信的CA授权。membership service provider作为一个fabric的组件，其中定义了如何治理这个org中的身份。默认的MSP使用X509证书，采用PKI分层模型。

## what is an MSP？

fabric是一个许可网络（permissioned network），任何接入网络的实体（identity）都需要由CA签发对应的证书。MSP的实现只需要一些文件夹，并将这些文件夹添加到网络中。这些文件夹定义一个org的内在的（org对peer的授权）和外在的（其他org验证该org中的peer被授权的）行为。

MSP通过已经获取的其他members的身份，或者已经被认证的CAs，来识别哪些Root CA和Intermediate CA已经被接收。

> The MSP identifies which Root CAs and Intermediate CAs are accepted to define the members of a trust domain by listing the identities of their members, or by identifying which CAs are authorized to issue valid identities for their members.

MSP不仅仅标识出了channel的成员或者网络的参与者，还为每个identity定义了一个role。这个role代表了一个identity在一个node或channel上的权限。例如创建一个peer就赋予peer权限。

## MSP domains

两种MSP最大的不同不是它们的功能，而是它们处于不同的范围。

### Local MSP

local MSP专门为client和nodes（peers和orderers）定义。被组织成一个文件系统的目录。

对于clients，MSP定义了他作为channel的成员是否有权利执行交易，或者作为特定的角色进入到系统中（例如configuration transactions）

对于nodes，每个node都必须有MSP，因为必须有管理权或者参与权。org，org管理员、node、node管理员都需要有相同的根信用。

### Channel MSPs

Channel MSP定义了在channel层面的管理和参与权限。Peers和ordering nodes在同一app channel中拥有统一的Channel MSP视图。<u>如果org希望加入该channel，则需要在channel configuration中包含该org peer信任链的MSP</u>。local MSPs需要添加到channel MSP的配置文件中。

- Channel MSP定义了在channel层面谁有权威性，定义了channel 成员和channel层面policy实施间的关系。Channel MSP包含了若干org的MSPs
- 每个参与到channel中的org都必须有一个MSP。org的MSP定义了哪个peer有权代表org执行操作。
- Channel MSP包含了所有在这个channel的org的MSP。
- Local MSP在对应的node或client上只有一份，而Channel MSP在channel的参与者都保存一份，并通过共识协议保持一致。

## What role does an organization play in an MSP？

一个org是一个逻辑概念，用于抽象一组members，通过MSP管理一组members。MSP允许一个实体链接到一个org。在简单的情况下，一个org可以设置一个MSP名字为 `ORG1-MSP`。但是在一些复杂的情况下，一个org可以拥有多个membership group，例如这个org在不同的orgs间执行不同的商业逻辑。例如，一个org可以有多个MSP，命名为 `ORG2-MSP-NATIONAL	`，`ORG2-MSP-GOVERNMENT`，分别对应于不同的信任链：NATIONAL对应售卖channel，GOVERNMENT对应监管channel。

### Organizational Units（OUs）and MSPs

一个org可以被分为多个职责不同的单元，可以被称为 `affiliations`。OU作为一个org内部的一个单元，例如在一个org1有 `ORG1.MANUFACTURING`和`ORG1.DISTRIBUTION`分别对应不同的业务线。在CA签发证书时，会在证书的 OU字段上填写该identity属于哪个OU。一个用OU字段的好处是可以通过该字段对identity进行一些基于属性的权限限制。

### Node OU roles and MSPs

> Node OU roles是在一个OU中为不同的node来分配角色，不同角色还得通过具体的policy来实现。

还有一个类在一个org中，组织多个node为一个 `Node OU`。通过设置node OU，可以在一个org中设置不同组的org，实现不同粒度的功能，例如在一个node OU中实现背书。为了使用Node OU功能，“identity classification”特性要保持开启。

一共有四种Node OU ROLES：

- client
- peer
- admin
- orderer

上面的四种MSP roles被填充到X509证书的 CommonName 上。但是赋予一个node角色不意味着他已经具有某种权利，角色的权利由具体的policy来确定。

OUs还有一个用处，就是区分不同orgs。在这种情况下，多个orgs使用在一个信任链上的root CA或者intermediate CA，但是不同的OU来区分不同的org，使得这些orgs更加中心化。

## MSP Structure

下图是一个local MSP的结构：

![MSP6](https://hyperledger-fabric.readthedocs.io/en/release-2.5/_images/membership.diagram.6.png)

- config.yaml

​	通过开启Node OUs，配置identity classification特性，

- cacerts

  受org信任的自签证书列表，必须包含至少一个根证书

- intermediatecacerts

  受org信任的中间证书列表，必须被根证书或者其他受信任的中间证书签发。一个中间证书代表着不同的org分区（例如，ORG1中的：ORG1-DISTRIBUTION和 ORG1-MANUFACTURING）

- keystore

  至少包括一个私钥

- signcerts

  被CA签署的证书。这个文件夹对于local MSP是必须的，因为它需要向别人证明自己的身份。但是对于channel MSP不是必须的，因为channel MSP仅仅需要提供身份验证的功能而不是签署的能力。

- tlscerts

  包含被org信任的root CA的自签证书，用于nodes间的tls通信

- operationscerts

  与fabric operation service通信

出了上面的文件之外，channel MSP还包括下面的内容：

- revoked certificates

  被org吊销的identity，类似与certificate revoked list（CRL）