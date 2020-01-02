# 系统架构 !heading

金融行业的区块链，不是公链，必须满足金融监管要求，因此在系统架构上就要充分考虑这个要求，本章节阐述满足金融业务的区块链系统架构应该注意的事项。

## 系统架构总览

![local image](../Images/02_system_architecture.png)

* 区域1：区块链网络，也是构建在互联网之上的一个虚拟网络
* 区域2/3/4：区块链网络中的节点区域，可以是单独的节点，也可以是节点和业务应用的混合区域；
  * DMZ：Dmilitarized zone,隔离区或者称为非军事化区域，位于两个防火墙之间，通常建议在这个区域部署区块链节点服务，连接区块链网络和内部业务网络；
  * SEAAPS: SEAAPS为区块链提供基础服务的应用，例如区块链浏览器，预言机服务；
  * Intranet/Service: 业务应用服务，是银行自己内部业务系统，例如银行账户系统，KYC服务等；
  * Regulator: 行业监管机构，对交易进行监管，满足反洗钱(AML)管理需要；
  * TAX: 税务监管机构；
* 三角形：代表区块链的共识节点，通常是联盟链管理机构进行选举指定的记账节点；
* 圆形：接入节点，联盟链许可接入的节点，没有记账权的节点

## 节点

SEAAPS区块链是由多个参与方的节点互相连接起来的一个虚拟的网络，网络中的每个参与方可以有一个或者多个节点，他们以网络服务的形式存在，每个节点都维护全部或者部分账本数据，整个区块链网络中账本数据有多个副本，因此单个或者部分节点失能，不会丢失数据，不会停止服务。

SEAAPS区块链的共识采用RANDOMIZED BFT的方式。SEAAPS区块链的核心有若干个验证节点维持系统的基本验证网络。SEAAPS区块链的验证网络对每一个接入SEAAPS区块链的应用开放。接入SEAAPS区块链的应用是指以SEAAPS区块链为平台的针对某些用户的应用程序。这些应用可以通过SEAAPS区块链提供的API直接接入SEAAPS区块链的公有区块链。这些应用可以起到一个验证节点的作用。

这样的节点可以实现两个功能:

1. 参与SEAAPS区块链网络的公共节点验证，实现应用接入SEAAPS区块链网络。
2. 如果应用只是仅仅使用API访问所需的区块链功能，则并不需要部署一个单独的验证节点，部署一个接入节点即可。

PBFT算法有三分之一的容错率，验证节点数量建议不低于6个节点。

## 共识算法

SEAAPS区块链技术采用自有知识产权的随机BFT共识算法。

PBFT的这个机制下有一个叫View的概念，在一个View 里，一个节点(REPLICA)会是主节点(PRIMARY)，其余的节点都叫备份节点(BACKUPS)。主节点负责将来自客户端的请求给排好序，然后按序发送给备份节点们。PBFT的这个主节点拥有比其它节点更加大的权利，如果它出现问题，会导致系统中比较大的延迟。在RBFT中，对这一点进行了改进，参考了RAFT中选举的机制，采用投票表决方式，无需抢夺记账权，保证各个节点权益的公平性。

## 金融区块链

金融区块链是一个联盟链，也是一个许可链。

金融区块链由金融行业中的企业构成的一个网络，通过这个网络来运行各种金融业务，其交易数据信息只有联盟内的成员才有资格查看，因此需要联盟成员进行批准才能接入联盟链。

监管部门（包括所在国税务机构）作为其中一个节点成员对交易实时监管，以满足反洗钱监管和其他业务监管要求，如果发生违反监管的交易，可以通过联盟链的治理机制对相关交易人钱包实施冻结，防止造成更大的违法后果。