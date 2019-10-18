BOS Oracle 部署文档
----------

# 1. 多签创建预言机相关账户

## 1.1. 导出创建账户的基础信息
`oracle.bos.json`
```
["eosio", "oracle.bos", {
      "threshold": 1,
      "keys": [],
      "accounts": [{
      "permission": {
      "actor": "eosio",
      "permission": "active"
     },
    "weight": 1
    }
  ],
  "waits": []
}, {
       "threshold": 1,
       "keys": [],
       "accounts": [{
       "permission": {
       "actor": "eosio",
       "permission": "active"
      },
     "weight": 1
   },{
     "permission": {
     "actor": "oracle.bos",
     "permission": "eosio.code"
    },
    "weight": 1
  }
 ],
 "waits": []
}]
```

## 1.2. 生成newaccount交易

```
cleos push action eosio newaccount ./oracle.bos.json -sdj -p eosio > oracle.trx
```
## 1.3. 生成buyram的交易

```
cleos system buyram -k eosio oracle.bos 3000 -sdj > oracle-ram.trx
```
## 1.4. 生成delegatebw的交易

```
cleos system delegatebw eosio oracle.bos "2 BOS" "2 BOS" -sdj > oracle-delegate.trx
```
## 1.5. 合并action到neworacle.json，发起提案

```
cleos multisig propose_trx createtest bppermission.json  neworacle.json  bostesterter
```
neworacle.json：
```
{
  "expiration": "2019-04-27T01:43:42",
  "ref_block_num": 33447,
  "ref_block_prefix": 2283537730,
  "max_net_usage_words": 0,
  "max_cpu_usage_ms": 0,
  "delay_sec": 0,
  "context_free_actions": [],
  "actions": [{
      "account": "eosio",
      "name": "newaccount",
      "authorization": [{
          "actor": "eosio",
          "permission": "active"
        }
      ],
      "data": "0000000000ea3055000000bdf79eb1ca0100000000010000000000ea305500000000a8ed32320100000100000000010000000000ea305500000000a8ed3232010000"
    },{
      "account": "eosio",
      "name": "buyrambytes",
      "authorization": [{
          "actor": "eosio",
          "permission": "active"
        }
      ],
      "data": "0000000000ea3055000000bdf79eb1ca00240000"
    },{
      "account": "eosio",
      "name": "delegatebw",
      "authorization": [{
          "actor": "eosio",
          "permission": "active"
        }
      ],
      "data": "0000000000ea3055000000bdf79eb1ca102700000000000004424f5300000000102700000000000004424f530000000000"
    }
  ],
  "transaction_extensions": [],
  "signatures": [],
  "context_free_data": []
}
```


# 2. 部署合约
## 2.1 生成 `set contract` 交易
```
cleos set contract oracle.bos bos.oracle/ -p oracle.bos  -s -j -d > setcontract.json
```
## 2.2 发起提案
```
cleos multisig propose_trx setcontract bppermission.json  setcontract.json  -p bostesterter
```
## 2.3 `approve` 多签
```
cleos multisig approve bostesterter updatasystem '{"actor":"${BP_NAME}","permission":"active"}' -p ${BP_NAME}
```
## 2.4 `exec` 执行
```
cleos multisig exec bostesterter setcontract -p bostesterter@active
```
## 2.5 初始化 Oracle
```
cleos push action ${contract_oracle} setparameter '{"version":1,"parameters":{"core_symbol":"BOS","precision":4,"min_service_stake_limit":1000,"min_appeal_stake_limit":200,"min_reg_arbitrator_stake_limit":10000,"arbitration_correct_rate":60,"round_limit":3,"arbi_timeout_value":86400,"arbi_freeze_stake_duration":259200,"time_deadline":86400,"clear_data_time_length":10800,"max_data_size":256,"min_provider_limit":1,"max_provider_limit":100,"min_update_cycle":1,"max_update_cycle":8640000,"min_duration":1,"max_duration":8640000,"min_acceptance":1,"max_acceptance":100}}' -p ${contract_oracle} -s -j -d  > setconfig.json

cleos multisig propose_trx setconfig bppermission.json setconfig.json  -p ${account}   

cleos multisig approve ${account} setconfig '{"actor":"${account}","permission":"active"}' -p ${account}

cleos multisig exec ${account} setconfig -p ${account}
```

_注：具体参数含义参看：[预言机全局参数配置](./BOS_Oracle_入门.md#27-预言机全局参数配置)


# 3. 导入仲裁员
## 3.1 生成 `importwps` 交易
```
cleos push action oracle.bos importwps '[["arbitrator11","arbitrator12","arbitrator13","arbitrator14","arbitrator15","arbitrator21","arbitrator22","arbitrator23","arbitrator24","arbitrator25", "arbitrator31", "arbitrator32","arbitrator33", "arbitrator34", "arbitrator35",  "arbitrator41",  "arbitrator42", "arbitrator43", "arbitrator44", "arbitrator45"]]' -p oracle.bos -s -j -d > importwps.json
```
## 3.2 发起提案
```
cleos multisig propose_trx insertwps  bppermission.json  importwps.json  -p bostesterter 
```
## 3.3 `approve` 并 `exec` 多签
```
cleos multisig approve  bostesterter insertwps '{"actor":"bostesterter","permission":"active"}'  -p  bostesterter
cleos multisig exec bostesterter insertwps -p bostesterter@active
```
## 
