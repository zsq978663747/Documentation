Token注册及管理
---------

由IBC通道连通的两条链是对等的，IBC支持将任一端链上资产转移到另一条链。
任何token必须在两条链上的跨链合约（ibc.token）中同时注册才能使用IBC通道。
当前BOSCORE团队提供的跨链合约（ibc.token）账户在EOS主网和BOS主网均为**bosibc.io**。
IBC系统为每个token提供了管理员功能，方便项目发管理token，因此项目方需要在EOS主网和BOS主网分别注册管理员账户.

为叙述方便，下文以注册EOS主网token为例进行说明，使其能够在BOS主网流通。


### 1. 对token合约的要求
1. transfer接口定义必须和`eosio.token`合约的`transfer`接口定义完全相同；
2. transfer函数中必须包含语句 `require_recipient( to )`；


### 2. 术语说明
- 正向转账  
  指EOS主网的Token通过IBC转移到BOS主网。
- 反向提现  
  指将BOS主网发行的原EOS主网的Token提现回EOS主网。


### 3. 项目方需要提供如下信息
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


### 3. 管理员权限
假设注册的某个token合约账户为"eosgoldtoken"，代币符号为"4,GOLD"。

*1. 在EOS主网`bosibc.io`合约中(table:`accepts`)*
管理员可以修改的Token相关参数包括:  

- 最大承兑数量  
  此数量应等于或小于原token合约的`max_supply`。  
  限制最大承兑数量的目的是降低安全风险。  
```bash
$ cleos push action bosibc.io setacptasset '["eosgoldtoken","max_accept","1000000000.0000 GOLD"]' -p ${eos_admin}
```
- 单次正向转账最小额度  
  注意：此额度必须大于IBC项目方设置的交易费额度。
```bash
$ cleos push action bosibc.io setacptasset '["eosgoldtoken","min_once_transfer","100.0000 GOLD"]' -p ${eos_admin}
```
- 单次正向转账最大额度  
```bash
$ cleos push action bosibc.io setacptasset '["eosgoldtoken","max_once_transfer","1000000.0000 GOLD"]' -p ${eos_admin}
```
- 每天正向转账最大额度  
```bash
$ cleos push action bosibc.io setacptasset '["eosgoldtoken","max_daily_transfer","10000000.0000 GOLD"]' -p ${eos_admin}
```
- 项目方名称  
```bash
$ cleos push action bosibc.io setacptstr '["eosgoldtoken","organization","organization name"]' -p ${eos_admin}
```
- 项目方官网  
```bash
$ cleos push action bosibc.io setacptstr '["eosgoldtoken","website","https://www.website.com"]' -p ${eos_admin}
```
- active状态  
  acitve状态为false时，合约将不接受此token的正向转账，但不影响反向提现
```bash
$ cleos push action bosibc.io setacptstr '["eosgoldtoken","active",true]' -p ${eos_admin}
```

*2. 在BOS主网`bosibc.io`合约中(table:`stats`)*
管理员可以修改的Token相关参数包括:  

- 最大承兑数量  
  此值应该和EOS主网`bosibc.io`合约中`最大承兑数量`值相同。
```bash
$ cleos push action bosibc.io setpegasset '["GOLD","max_supply","1000000000.0000 GOLD"]' -p ${bos_admin}
```
- 单次反向提现最小额度  
```bash
$ cleos push action bosibc.io setpegasset '["GOLD","min_once_withdraw","100.0000 GOLD"]' -p ${bos_admin}
```
- 单次反向提现最大额度  
```bash
$ cleos push action bosibc.io setpegasset '["GOLD","max_once_withdraw","1000000.0000 GOLD"]' -p ${bos_admin}
```
- 每天反向提现最大额度  
```bash
$ cleos push action bosibc.io setpegasset '["GOLD","max_daily_withdraw","10000000.0000 GOLD"]' -p ${bos_admin}
```
- active状态  
  acitve状态为false时，合约将不接受此token的反向提现，但不影响正向转账
```bash
$ cleos push action bosibc.io setpegbool '["GOLD","active",true]' -p ${bos_admin}
```
