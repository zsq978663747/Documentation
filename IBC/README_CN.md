# IBC 用户协议
## 简介

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


## Memo Schema 

转账Memo格式:
```
    ACCOUNTNAME@CHAINNAME TRANSFER_MESSAGE
    账户名@链名 转账备注
```

举例：

- 从 EOS主网账户`eosuser1`转账100 EOS 到 BOS主网 `bosuser2`
```
    cleos transfer eosuser1 bosibc.io "100.0000 EOS" "bosuser2@bos 这是一笔EOS到BOS的跨链转账"
```

- 从 BOS主网账户`bosuser2`转账100 EOS 到 EOS主网 `eosuser1`
```
    cleos transfer bosuser2 bosibc.io "100.0000 EOS" "eosuser1@eos 这是一笔BOS到EOS的跨链转账"
``` 

*说明：由于进行的跨链转账，所以到账时间会有延迟，处于安全考虑，等待两端交易都进LIB，时间应该在 6分钟左右*


因此用户使用现有的手机app钱包即可完成跨链资产，但需要现有的钱包增加支持ibc.token合约，支持也很简单，因为ibc.token合约的transfer接口定义和eosio.token的transfer定义完全相同，手机app钱包也可以提供专有的的界面，增强用户体验。









