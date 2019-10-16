BOS Oracle Deployment
----------

# 1. Create Oracle Contract Account `oracle.bos`

## 1.1. Export the basic info the account
oracle.bos.json
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

## 1.2. Create `newaccount` ACTION

```
cleos push action eosio newaccount ./oracle.bos.json -sdj -p eosio > oracle.trx
```
## 1.3. Generate `buyram` ACTION

```
cleos system buyram -k eosio oracle.bos 3000 -sdj > oracle-ram.trx
```
## 1.4. Create `delegatebw` ACTION

```
cleos system delegatebw eosio oracle.bos "2 BOS" "2 BOS" -sdj > oracle-delegate.trx
```
## 1.5. Combine ACTIONs into `neworacle.json`，create proposal

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
# 2. Set contract
## 2.1 Create `set contract` ACTION
```
cleos set contract oracle.bos bos.oracle/ -p oracle.bos  -s -j -d > setcontract.json
```
## 2.2 Create MSIG
```
cleos multisig propose_trx setcontract bppermission.json  setcontract.json  -p bostesterter
```
## 2.3 BPs `approve` MSIG
```
cleos multisig approve bostesterter updatasystem '{"actor":"${BP_NAME}","permission":"active"}' -p ${BP_NAME}
```
## 2.4 `exec` execute MSIG
```
cleos multisig exec bostesterter setcontract -p bostesterter@active
```


# 3. Import Arbitrators
## 3.1 Create `importwps` ACTION
```
cleos push action oracle.bos importwps '[["arbitrator11","arbitrator12","arbitrator13","arbitrator14","arbitrator15","arbitrator21","arbitrator22","arbitrator23","arbitrator24","arbitrator25", "arbitrator31", "arbitrator32","arbitrator33", "arbitrator34", "arbitrator35",  "arbitrator41",  "arbitrator42", "arbitrator43", "arbitrator44", "arbitrator45"]]' -p oracle.bos -s -j -d > importwps.json
```
## 3.2 Create MSIG
```
cleos multisig propose_trx insertwps  bppermission.json  importwps.json  -p bostesterter 
```
## 3.3 BPs `approve` MSIG
```
cleos multisig approve  bostesterter insertwps '{"actor":"bostesterter","permission":"active"}'  -p  bostesterter
```

##  3.4 `exec` MSIG

```
cleos multisig exec bostesterter insertwps -p bostesterter@active
```

