IBC User Guide
-------

### 1. Preface
On the two blockchains between which realized inter-blockchain communication, 
any token conforming to the `eosio.token specification` can register and use the IBC channel for inter-blockchain transfer.
This article describes the IBC user interface and provides command line examples.


### 2. "transfer" action 
source code:
``` 
    [[eosio::action]]
    void transfer( name from, name to, asset quantity, string memo );
```

*1. "to" account*
The "to" account must be the account which deploied ibc.token contract. In BOS-EOS IBC system, the "to" account on 
both EOS mainnet and BOS mainnet are **bosibc.io**.

*2. "quantity"*
The token must be registered, and the quantity amount must satisfies this token's constraints, 

*3. "memo" format*
memo format for ibc transaction is very important, because you should provide acceptance accounts on peer chain in memo string. format is:
```
    {account_name}@{chain_name} {user-defined string}
 
    {user-defined string} is optinal
    {chain_name} is defined by "peerchain_name" in ibc.token's "globals" table.
    
    examples:
    'bosaccount11@bos happy new year 2019'
    'eosaccount11@eos'
```

note: if you want to transfer token to "to" account itself, not want a ibc transaction, the memo string must star
with "local", otherwise the transaction will fail. source code refer `token::transfer_notify()` and ` token::transfer()`
in ibc.token contract.


### 3. Command Line Examples
Transfer 100 EOS from EOS mainnet account `eosaccount` to BOS mainnet `bosaccount`
```
    cleos -u <eos-mainnet-api> transfer eosaccount bosibc.io "100.0000 EOS" "bosaccount@bos hello!"
```

Withdraw 100 EOS from BOS mainnet account `bosaccount` to EOS mainnet `eosaccount2`
```
    cleos -u <bos-mainnet-api> transfer -c bosibc.io bosaccount bosibc.io "100.0000 EOS" "eosaccount2@eos hi!"
``` 

After send transfer action, and waiting for 4 to 5 minutes, you can go to the peer chains to check if you have received the token.

So users can transfer assets across the chains by using any existing mobile app eosio wallets, 
the existing wallets only need to support the ibc.token contract, because the transfer action interface definition of ibc.token 
contract is exactly the same as that of eosio.token contract


### 4. Token Quotas
All token quotas are defined in ibc.token contracts, take bosibc.io as an example of ibc.token contract, 
you can get them by following command:
``` 
cleos -u <eos-mainnet-api> get table bosibc.io bosibc.io accepts
cleos -u <eos-mainnet-api> get table bosibc.io bosibc.io stats
cleos -u <bos-mainnet-api> get table bosibc.io bosibc.io accepts
cleos -u <bos-mainnet-api> get table bosibc.io bosibc.io stats
```
