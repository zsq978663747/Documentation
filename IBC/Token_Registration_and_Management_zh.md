
如何注册EOS主网token使用IBC通道在BOS主网流通
-----



当将一条链上的token信息向本链和对方链的ibc.token合约注册此token信息后，ibc系统开始接受对此token的跨链交易。ibc系统支持两端链上任意多种token的跨链需求。

ibc系统在一条链上只有一个ibc.token合约，原链上被映射的多种token都会在本合约中进行管理。

对需要跨链的token合约的需求：
- 第一：其transfer接口必须和 “eosio.token”合约的transfer接口定义完全相同
- 第二：其transfer函数中必须包含 require_recipient( to ); 语句

在一个token被注册后
- eosio用户调用被承接token的transfer接口并提供适当的memo信息，即可完成资产从原链到token映射链的转移。
- eosio用户调用ibc.token合约transfer接口并提供适当的memo信息，即可完成映射资产转回原链的操作。

## 跨链合约账户

为了保证统一的用户使用体验，在EOS主网和BOS主网上同时都会使用 `bosibc.io` 这个账户来部署，用户可以通过在 `memo` 里面指定是那条链。

*注: 感谢 StartEOS 团队的贡献的 EOS主网账户 `bosibc.io`*






### 术语说明
- 正向转账  
  指EOS主网的Token通过IBC转移到BOS主网。
- 反向提现  
  指将BOS主网发行的原EOS主网的Token提现回EOS主网。

### 简述
EOS主网的token需要在EOS主网的`bosibc.io`和BOS主网的`bosibc.io`合约中分别注册token的信息才可以使用IBC通道。  
IBC系统为每个token提供了管理员功能，项目方需要在EOS主网和BOS主网分别注册管理员账户.  

### 管理员权限
在EOS主网`bosibc.io`合约中(table:`accepts`)，管理员的可以修改此Token相关参数包括:  
- 最大承兑数量  
  此数量应等于或小于原token合约的`max_supply`。  
  限制最大承兑数量的目的是降低安全风险。  
- 单次正向转账最小额度  
- 单次正向转账最大额度  
- 每天正向转账最大额度  
- 项目方名称  
- 项目方官网  
- active状态  
  acitve状态为false时，合约将不接受此token的正向转账，但不影响反向提现

在BOS主网`bosibc.io`合约中(table:`stats`)，管理员的可以修改此Token相关参数包括:  
- 最大承兑数量  
  此值应该和EOS主网`bosibc.io`合约中`最大承兑数量`值相同。
- 单次反向提现最小额度  
- 单次反向提现最大额度  
- 每天反向提现最大额度  
- active状态  
  acitve状态为false时，合约将不接受此token的反向提现，但不影响正向转账


### 项目方需要提供如下信息
  
- 项目方名称  
- 项目方官网  
- 在EOS主网原Token合约账户
- Token的symbol  
  包括符号和精度
- 准备在BOS主网使用的锚定币symbol  
  包括符号和精度，（即，在BOS上此Token的锚定币可以使用和原Token相同的符号，也可以使用不同的符号）
- 在EOS主网管理员账户  
- 在BOS主网管理员账户  
- 最大的承兑数量  
- 单次正向转账最小额度  
- 单次正向转账最大额度  
- 每天正向转账最大额度  
- 单次反向提现最小额度  
- 单次反向提现最大额度  
- 每天反向提现最大额度  
- 初始active状态  

项目提供上述信息并经BOSCORE基金会审核，审核通过后，BOCCORE团队会发起多签在两条链上注册Token相关信息，
注册完成后，此token可开始使用IBC通道。
