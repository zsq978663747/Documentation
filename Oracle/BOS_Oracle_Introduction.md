BOS.Oracle Introduction: Witness and Changes
----------

# I, Background

The blockchain technology has evolved tremendously over the past few years and is widely known by public now, however there are not many use cases. The main reason is that the blockchain is in a closed system and cannot communicate with external world in a **trusted manner**. If we can solve this trust problem, there will be much more use cases for the blockchain technology.

In the real world, if a merchant is not sure that he can receive the money, he will not sell the product. If the consumer cannot determine the authenticity of the product, he will not purchase it. "

"Trusted" is the most basic condition of trading in the real world. If we can't foster trust, then we can't complete the transaction. Trust can either rely on the credibility of both parties (which is essentially based on past transaction history, making probabilistic estimates) or rely on a certain The third party of the letter (whether it is really trustworthy, where does the basis of trust come from?). In complex real-world, foster allows us to take on risks and consume energy and resources significantly. How to solve this problem? Can we combine blockchain technology to change this situation? BOS's oracle (Oracle) service is our answer, hoping to serve as a data bridge between the blockchain and the real world.

Some oracles are designed based on the assumption of a trusted data source or an authoritative data source. however this hypothesis is theoretically very risky and cannot guarantee the authenticity of the data source. The principles that BOS's oracle system follows from the beginning of its construction are:

    BOS Oracle does not relying on each oracle provider's 100% credibility of the data they provided. 
    BOS Oracle treat each oracle provider as a participant in the game, in order to achieve overall credibility in the game.

In this way, as long as the participants and the real world roles are mapped in the game, not only we can  obtain the trustworthiness of the blockchain input data , but also we can output "trust" to the real world. In fact, this is more like a trusted platform based on blockchain, and its service is a predictor. The BOS oracle will extend the value of the blockchain from its monetary attributes to the construction of transactions and rules. This extension will solve or improve many real-world trust problems, thereby expanding the application boundaries of the blockchain and ultimately let blockchain technology real-world scenarios other than transaction transfers.

# II. Introduction

## 2.1 System Components
![Structures](https://raw.githubusercontent.com/boscore/Documentation/master/imgs/oralce/bosoracle_structure.png)

In order to cope with various situations and conflicts from different levels of oracle service, BOS Oracle use “**two-way proof**” to obtain final arbitration results through mechanisms such as “**different arbitrators**” and “**multiple rounds of arbitration**”. In the arbitration, the mortgage mechanism is adopted as the final penalty anchor, the appeal system and the prosecution system are carried out simultaneously, mutual supervision; the arbitrator is stimulated by means of income sharing and punishment as a reward, and the credit system is introduced to realize the model under the non-one-time game. Stable and trustworthy.

**Service:** Provide Data, Fetch Data, Arbitration

**Role:** Provider, User, DAPP, Arbitrator

**Features**: Comprehensive arbitration, holistic check and balance, systematic incentive, wide-range of application

## 2.2 Data Service

Service Provider can create data services to upload data, and finally, the value of the data will be realized in the end.

## 2.3 Use Oracle Service

The data user can use an existing oracle service according to his own needs, or can participate as an initiator in the corresponding oracle service and obtain the required data from the service.

Users can easily query and retrieve the corresponding oracle service in the Oracle Store.

## 2.4 Arbitration

The arbitration module is the backbone for fairness. BOS Oracle adopted a number of rounds of arbitration in order to deter bribery, and the number of arbitrators increase will commensurate with each arbitration round. As the size of the arbitrators expand, it will become more difficult to pay bribes, and the cost will increase sharply, which will eventually prevent, and deter corrupt activities.

The arbitration model of the BOS Oracle is divided into two phases:

- The "Fulltime Arbitrators" phase, 21 arbitrators in total who must be timely response to the arbitration cases; in this stage, a total of 3 rounds of arbitration can be conducted, with 3, 5 and 9 persons in each round and no repetition in each round;
- The "Mass Arbitrators” phase, the participants in the entire Oracle service will be involved in. Extensive arbitration and random sampling will be conducted, subject to the majority; The number of "Mass Arbitrators" will be well regulated, in case of "Sybil Attack".

![bosoraclegamemodel](https://raw.githubusercontent.com/boscore/Documentation/master/imgs/oralce/bosoracle_game_model.png)  

## 2.5 Intelligent Risk Control

In order to mitigate risk, “intelligent risk control” always ensures that the data provider has sufficient collateral tokens. DApp user to transfer Token directly to the "smart-controlled" account instead of the DApp project's account.

When someone initiates arbitration for the oracle service, "Intelligent Risk Control" will calculate fund freezing time and the thawing duration based on historical behaivor and data provide. If there is no anomaly in the entire process, the DApp can extract the funds from the “Intelligent Risk Control” account.

intelligently based on the historical behavior of the data provider and the current expenditure. 

“Intelligent Risk Control” is an option that depends on whether the DApp project party is adopting it. However, from the perspective of market competition, DApps that provide more protection for users will gain more favor.

![智能风控](https://raw.githubusercontent.com/boscore/Documentation/master/imgs/oralce/bosoracle_risk_control.png)  

## 2.6 Variable Name

- `${contract_oracle}`:  BOS Oracle Account
- `${provider_account}`:  Data Provider


# III, Data Provider

_Details will be released on Oracle Store_


## 3.1 Register Service

Anyone can create a data service and set the service basic parameters by the creator. After the data service is created, anyone can join and become the data provider by transfer pledge. The **mortgage** is required for registration, and **the mortgage is not lower than the service set mortgage** for the data provider to initiate the service.

“Intelligent Risk Control” is an option that depends on whether the DApp project party is adopting it. However, from the perspective of market competition, DApps that provide more protection for users will gain more favor....

```
cleos push action ${contract_oracle} regservice '{"account":"${create_account}","base_stake_amount":"1000.0000 BOS","data_format":"json", "data_type":0, "criteria":"Subject to majority", "acceptance":51, "declaration":"BTC Price", "injection_method":0, "duration":500,"provider_limit":3, "update_cycle":600}' -p ${account}
```

**Parameters:**

- `account`: Account which register the service
- `base_stake_amount`: Minimum stake amount for provider
- `data_format`: Data format: text, json, custom, if the format is json, input should be`{"xxx":"xxxx"}`
- `data_type`: Data type, 0 is deterministic, 1 is non-deterministic
- `criteria`: Service guidelines, when someone lodges a complaint against the service, the arbitrator will refer to the evidence and service guidelines.
- `acceptance`: This number is the threshold for the provider's data to be valid. For example, there are 10 data providers, 51 means that at least 5 provide the same data.
- `declaration`: Service statement and description
- `injection_method`: Injection mode, 0 is written to the table, and subsequent versions implement other methods.
- `duration`: Data Collection Duration, in seconds
- `provider_limit`: The minimum threshold for the number of data providers required to open the data service
- `update_cycle` : Update cycle, next  `cycle_number`, in seconds

## 3.2 Join Service

First, find the oracle service

```
cleos get table ${contract_oracle} ${contract_oracle} dataservices 
```

Find Oracle and transfer/stake EOS to register corresponding service

```
cleos transfer ${provider_account} ${contract_oracle} "1000.0000 BOS" "0,${service_id}"  
```

If the creator does not find the service to be registered, you can register the service before joining.

**Transfer and Stake memo:**

- `num`:**0 is to join service**
- `service_id`: Service ID

### 3.2.1 Service Example

**Step 1: Check All Services**

```
cleos get table ${contract_oracle} ${contract_oracle} dataservices
```

Result: 
```
{
  "rows": [{
      "service_id": 1,
      "data_type": 0,
      "status": 1,
      "injection_method": 0,
      "acceptance": 3,
      "duration": 3000,
      "provider_limit": 3,
      "update_cycle": 3600,
      "last_update_number": 0,
      "appeal_freeze_period": 0,
      "exceeded_risk_control_freeze_period": 0,
      "guarantee_id": 0,
      "base_stake_amount": "1000.0000 BOS",
      "risk_control_amount": "0.0000 BOS",
      "pause_service_stake_amount": "0.0000 BOS",
      "data_format": "json",
      "criteria": "Subject to majority",
      "declaration": "BTC Price"
    },{
      "service_id": 2,
      "data_type": 0,
      "status": 0,
      "injection_method": 0,
      "acceptance": 3,
      "duration": 3000,
      "provider_limit": 3,
      "update_cycle": 3600,
      "last_update_number": 0,
      "appeal_freeze_period": 0,
      "exceeded_risk_control_freeze_period": 0,
      "guarantee_id": 0,
      "base_stake_amount": "1000.0000 BOS",
      "risk_control_amount": "0.0000 BOS",
      "pause_service_stake_amount": "0.0000 BOS",
      "data_format": "{\"data\":\"2019-05-07\",\"time\":\"12:22:21\"}",
      "criteria": "Subject to majority",
      "declaration": "Oil Price",
      "update_start_time": "2019-09-19T03:00:00"
    }
  ],
  "more": false
}
```

**Step 2: Join the service_id 1**

```
cleos transfer oracleprovi1 ${contract_oracle} "1000.0000 BOS" "0,1"
```

**Step 3: Find `oracleprovi1` through command below**

```
cleos get table ${contract_oracle} 1 svcprovision
```

Result: 
```
{
  "rows": [{
      "service_id": 1,
      "account": "oracleprovi1",
      "amount": "1000.0000 BOS",
      "freeze_amount": "0.0000 BOS",der
      "service_income": "0.0000 BOS",
      "status": 0,
      "public_information": "",
      "create_time": "2019-09-26T02:30:31"
    }
  ],
  "more": false
}
```


## 3.3 Provider Reduce Stakes

Provider can reduce service stakes, but no less than the provider_limit

```
cleos push action ${contract_oracle} unstakeasset '{"service_id":${service_id},"account":"${provider_account}","amount":"1.0000 BOS","memo":"unstake mount"}' -p ${provider_account} 
```

**Parameters:**

- `service_id`: Service ID
- `provider_account`: Data Provider Account
- `amount`: unstake amount
- `memo`: memo

## 3.4 Push Data

After the oracle machine service is successfully added, the data will be uploaded: upload

```
cleos push action ${contract_oracle} pushdata '{"service_id":1,"provider":"${provider_account} ","cycle_number":"3","request_id":0,"data":"1 test data json 3 "}' -p ${provider_account}
```

**Parameters:**
- `service_id`: Service ID
- `provider_account`: Provider
- `cycle_number`: the number is calculated according to the `update_cycle` of the service, and the calculation formula `cycle_number=current seconds/update_cycle`; when each round exceeds duration, the data is not allowed to be uploaded.
- `request_id`: The id of the data user when the request is initiated, and the data provider needs to correspond to upload data, in v1.0 `request_id` is 0
- `data`: In order to upload the content of the data, all data providers must upload the data according to the provisions of the service, otherwise the upload will fail.


# IV, Dapp

Dapp can obtain data through `service_id`. If the service is deterministic, we can only obtain one data for each `cycle_number` ; If the service is an non-deterministic data type, each `cycle_number` of the service can see multiple pieces of data, and the user needs to select according to his own needs.

```
cleos get table ${contract_oracle} ${service_id} oracledata
```

**Parameters:**
- `service_id`: Service ID

## 4.1 Deterministic Data: 
Take the lottery draw result as an example, Service ID = 5 is the final result
```
cleos get table ${contract_oracle} 5 oracledata
{
  "rows": [{
      "record_id": 0,
      "request_id": 0,
      "cycle_number": 2,
      "data": "02 03 01 17 12 04 05",
      "timestamp": 1569554558
    },{
      "record_id": 1,
      "request_id": 0,
      "cycle_number": 4,
      "data": "02 12 34 11 02 04 15",
      "timestamp": 1569563694
    }
  ],
  "more": false
}
```

## 4.2 Non-deterministic Data: 
Take BTC Price as an example, to find Service ID is 7, and obtain the current BTC price: 
```
cleos get table ${contract_oracle} 7 oracledata
```

result: 
```
{
  "rows": [{
      "record_id": 0,
      "request_id": 0,
      "cycle_number": 1,
      "data": "time:2019-09-27T03:27:33.216857+00:00,price:8901 USDT",
      "timestamp": 1569552869
    },{
      "record_id": 1,
      "request_id": 0,
      "cycle_number": 1,
      "data": "time:2019-09-27T03:29:33.216857+00:00,price:8911 USDT",
      "timestamp": 1569552932
    },{
      "record_id": 2,
      "request_id": 0,
      "cycle_number": 1,
      "data": "time:2019-09-27T03:29:33.216857+00:00,price:8910 USDT",
      "timestamp": 1569552965
    }
  ],
  "more": false
}
```

# V, Arbitration

## 5.1 Arbitrator Registration  (mortgage)

The first round of arbitrators will appointed 21 designated BOS WPS full-time auditors; in the later period, open community registration, in order to ensure that the arbitration is effective and fair, the arbitrators must vote through the community to meet the conditions to be eligible. The minimum deposit at registration is 100000 BOS.

Transfer Mortgage Registration Arbitrator:

```
cleos transfer ${arbitrator_account} ${contract_oracle} "100000.0000 BOS" "4,1" -p ${arbitrator_account}  
```

**Transfer and Stake memo:**
- `${arbitrator_account}`: Arbitration Account
- `num`:**4 is arbitrator registration**
- `arbitrat_type`: 1 is full-time arbitrator, 2 is public arbitrator

## 5.2 Arbitration Commence

If the user or DAPP party is in use, if there is any problem with the service data, you can apply for arbitration.

- If the respondent is not accepted, the system will determine that there is no response and the respondent lose the corresponding deposit, and if it is the data provider, the mortgage deposit will be deducted.
- If the arbitrator does not respond, the system will re-select the appropriate number of remaining arbitrators to send the invitation notice.
- The arbitration process will send a notice to the relevant party's account by transfer, and the Oracle Store will also push the notification through the registered channel.

There are 9 arbitration steps: 

1. `arbi_init` arbitration initiated
2. `arbi_choosing_arbitrator` select arbitrator
3. `arbi_wait_for_resp_appeal` wait for respond
4. `arbi_wait_for_accept_arbitrate_invitation` wait arbitrator to accept invitation
5. `arbi_wait_for_upload_result` wait arbitrator to upload result
6. `arbi_wait_for_reappeal` wait and reappeal
7. `arbi_resp_appeal_timeout_end` respond timeout
8. `arbi_reappeal_timeout_end` reappeal timeout
9. `arbi_public_end`  publish the result


Query current status through `arbitration_id`: 

```
cleos get table ${contract_oracle} ${arbitration_id} arbicase
```

**Parameter:** 

- `arbitration_id`: Arbitration ID
- `arbi_step`: Status of the Arbitration Case
- `last_round`: Last Round 
- `final_result`: 1 claimant win, 2 respondent win
- `arbitrators`:  list of available arbitrators
- `chosen_arbitrators`:  list of chosen arbitrators

Check  `arbitration_id=4`: 

```
cleos get table ${contract_oracle} 4 arbicase
```

Result: 

```
{
  "rows": [{
      "arbitration_id": 4,
      "arbi_step": 6,
      "arbitration_result": 2,
      "last_round": 2,
      "final_result": 0,
      "answer_arbitrators": [
        "arbitrator12",
        "arbitrator14",
        "arbitrator35",
        "arbitrator23",
        "arbitrator15",
        "arbitrator42",
        "arbitrator22",
        "arbitrator13"
      ],
      "chosen_arbitrators": [
        "arbitrator12",
        "arbitrator35",
        "arbitrator14",
        "arbitrator23",
        "arbitrator15",
        "arbitrator42",
        "arbitrator22",
        "arbitrator13"
      ]
    }
  ],
  "more": false
}
```

### 5.2.1 Account Initiate Appeal

If the data user or data provider finds that someone has cheated and provided false information, they can initiate an appeal; the first mortgage amount must not be less than 200 BOS at the time of appeal, and the subsequent mortgages must not be less than: `200*2^(N-1), N is round number`。

Transfer mortgage to initiate an appeal:

```
cleos transfer ${initiator_account} ${contract_oracle} "200.0000 BOS" "3,${service_id},'evidence','info','reason'" -p ${initiator_account} 
```

**Transfer and Stake memo:**

- `initiator_account`: Initiator Account
- `num`:**3 = Appeal**
- `service_id`: Service ID
- `evidence`: Evidence submitted at the time of appeal, recommended as IPFS file address
- `info`: Public information
- `reason`: reason for appeal

After the appeal order is issued, it will be notified by `transfer` ACTION, and all respondents will be notified. if any account should respond (including accounts other than the notice), and the case will be established.

### 5.2.2 Responding to the accounts

When someone complains, the relevant personnel will receive the notice of the appeal. At this time, the responding order needs to be executed to inform the other party to accept the appeal. At this time, the arbitration process is officially opened; if no one responds, after the timeout, the complainant will be directly judged as the winner. If the data provider is the losing party, all the mortgage payments for the data service will be deducted and distributed to the claimant.

The first responding mortgage is at least 200 BOS, and the subsequent mortgages must not be less than: 200*2^(N-1), N is the number of rounds.

```
cleos transfer ${accept_account} ${contract_oracle} "200.0000 BOS" "5,${arbitrat_id},'evidence'" -p ${accept_account} 
```

**Transfer and Stake memo:**

- `accept_account`: respondent account
- `num`: ,**5 is response**
- `arbitrat_id`: Arbitration ID
- `evidence`: Evidence submitted at the time of appeal, recommended as IPFS file address

### 5.2.3 Arbitrator Accepts Invitation

When the data provider responds, it will send an arbitration invitation to the arbitrator. Only `2^N+` arbitrators can accept the invitation before proceeding to the next step: Upload the arbitration result. If an arbitrator accepts the invitation over time, it will re-select from the remaining arbitrators until `2^N+1` arbitrators accept all, `N` is the number of rounds.

Arbitrator accept invitation:

```
cleos push action ${contract_oracle} acceptarbi '["${arbitrator}",${arbitrat_id}]' -p ${arbitrator}@active 
```

**Parameters:**

- `arbitrator`: Arbitrator
- `arbitrat_id`: Arbitration ID

### 5.2.4 Arbitrator Uploads Result

Arbitrator upload result: 

```
cleos push action ${contract_oracle} uploadresult '["${arbitrator}",${arbitrat_id}, ${result},'memo']' -p {arbitrator}@active
```

**Parameters:**

- `arbitrator`: Arbitration Account
- `arbitrat_id`: Arbitration ID
- `result`:  1 is claimant win, 2 is respondent win
- `memo`:  include the IPFS address of the arbitration process

### 5.2.5 End of Arbitration

Two ways to end arbitration process:

- After the arbitration result is uploaded, if no one initiates the appeal again, the arbitration will be closed after 72 hours;
- In version 1.0, only `full-time arbitrators` are supported, claimant can have up to three rounds of arbitration

After the arbitration is over, the arbitration will result in the final result, and the relevant account will be rewarded/punished.

## 5.3 Arbitration stakeholders can claim rewards 

After the arbitration case is over, the relevant personnel involved in the arbitration can be rewarded because the arbitration process involves three different roles. The claimant, the respondent, the arbitrator, and the distribution of income for different roles are also different. To receive the rewards from arbitration, stakeholders should call action `claim` to claim reward

- Arbitrator: The reward is derived from**loser's mortgage**
- Data users: If successful, all**service mortgage of the data provider** will be distributed equally to all appeal accounts, and the data users' appeal mortgage will be refunded.
- Data provider: If successful, the appeal mortgage will be refunded
- If the data provider sues other data providers, the winner is divided into the other party's**service mortgage**


View Arbitration Income:

```
cleos get table ${contract_oracle} ${account} arbiincomes
```

Claim income:

```
cleos push action ${contract_oracle} claimarbi '["${account}","${receive_account}"]' -p ${account}
```

**Parameters:**
- `account`: account related to the arbitration
- `receive_account`: account which receive the arbitration rewards

## 5.4 Unstaking Arbitration Mortgage

After the arbitration is over, the winner can query the mortgage amount from table and unstake the mortgage.

**1. Query the current mortgage number of the arbitration:**

```
cleos get table ${contract_oracle} ${arbitrat_id} arbistakeacc
```

**Parameters:**

`arbitrat_id`: Arbitration ID,  if the arbitration is end, the value will be `uint64_t(1)<<63 + arbitrat_id`

**2. Unstaking:**

```
cleos push action ${contract_oracle} unstakearbi '{"arbitration_id":${arbitrat_id},"account":"${account}","amount":"100.0000 BOS","memo":"unstake mount"}' -p ${account} 
```

**Parameters:**
- `arbitrat_id`: Arbitration ID
- `account`: account for unstaking
- `amount`: The number of tokens to be mortgaged(cannot be greater than the number of mortgages)
- `memo`: memo

# V, Transfer and Stake Memo Explanation

memo : `"Service Type,Service ID,……."`

**1. Register Service:**
memo : `"0,Service ID"`

```
cleos transfer ${provider_account} ${contract_oracle} "1000.0000 BOS" "0,${service_id}" 
```

**2. Appeal Service:**
memo : `"3,Service ID,evidence,public info,appeal reason"`

```
cleos transfer ${initiator_account} ${contract_oracle} "200.0000 BOS" "3,${service_id},'evidence','info','reason'" -p ${initiator_account}    
```

**3. Register Arbitrator:**
memo : `"4,Service ID"`

```
cleos transfer ${arbitrator_account}  ${contract_oracle} "10000.0000 BOS" "4,1" -p ${arbitrator_account}  
```

**4. Respond to Appeal:**
memo : `"5,Service ID,evidence"`

```
cleos transfer ${accept_account}  ${contract_oracle} "200.0000 BOS" "5,${service_id},'evidence'" -p ${accept_account} 
```


# VII, Oracle tables

**1. Query all current service information**

```
cleos get table ${contract_oracle} ${contract_oracle} dataservices
```

**2. Query the details of a service**

```
cleos get table ${contract_oracle} ${service_id} svcprovision
```

**3. Query all data display of a service**

```
cleos get table ${contract_oracle} ${service_id} oracledata
```

**4. Query all current data provider information**

```
cleos get table ${contract_oracle} ${contract_oracle} providers
```

**5. Query all current arbitrator information**

```
cleos get table ${contract_oracle} ${contract_oracle} arbitrators
```

**6. Query the progress of an arbitration case**

```
cleos get table ${contract_oracle} ${arbitrat_id} arbicase
```

**7. Query the Arbitration Evidence**

```
cleos get table ${contract_oracle} ${arbitrat_id} arbievidence
```

**8. Query the mortgage of an ongoing arbitration case**

```
cleos get table ${contract_oracle} ${arbitrat_id} arbistakeacc
```

**9. Query the information of a closed arbitration case**

```
cleos get table ${contract_oracle}  ${9223372036854775808+arbitrat_id} arbistakeacc
```

**10. View Arbitration Income Statement**

```
cleos get table ${contract_oracle} ${account} arbiincomes
```
