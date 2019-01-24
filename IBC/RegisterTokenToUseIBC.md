
如何注册EOS主网token使用IBC通道在BOS主网流通
-----

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


