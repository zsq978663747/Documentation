
IBC System Deployment
-----

This article will detail how to build, deploy and use the IBC system, and take the deployment between `Kylin testnet`
and the `BOS testnet` as an example. If you want to deploy IBC system between EOSIO chains whose source code unmodified, 
such as between `EOS mainnet` and `Kylin or Jungle testnet`, you need not `ibc_plugin_bos`, 
only `ibc_plugin_eos` is needed; if you want to deploy between EOSIO chains whose underlying source code has been modified,
it's need to determine whether those chains can use the `ibc_contracts` and `ibc_plugin` according to the actual situation.

### Build
- Build contracts  
  Please refer to [ibc_contracts](https://github.com/boscore/ibc_contracts).
  
- Build ibc_plugin_eos and ibc_plugin_bos  
  Please refer to [ibc_plugin_eos](https://github.com/boscore/ibc_plugin_eos) and [ibc_plugin_bos](https://github.com/boscore/ibc_plugin_bos).
  
### Create accounts
Three accounts need to be registered on each chain， one relay account for ibc_plugin, and two contract accounts 
for deploying IBC contracts, let's assume that the three accounts are:   
**ibc3relay333**: used by ibc_plugin, and need a certain amount of cpu and net.   
**ibc3chain333**: used to deploy ibc.chain contract, and need a certain amount of ram, you can buy 5 to 10 megabytes for it.  
**ibc3token333**: used to deploy ibc.token contract, and need a certain amount of ram, you can buy 5 to 10 megabytes for it.  
  
### Configure and start ibc_plugin
Suppose that the IP address and port of the relay node of Kylin testnet are 192.168.1.101:5678, node of BOS testnet are
192.168.1.102:5678

Some key configurations and ibc-related configurations are listed below. 

- configure relay node of Kylin testnet  
``` 
max-transaction-time = 1000
contracts-console = true

# ------------ ibc related configures ------------ #
plugin = eosio::ibc::ibc_plugin

ibc-listen-endpoint = 0.0.0.0:5678
#ibc-peer-address = 192.168.1.102:5678 
ibc-max-nodes-per-host = 5

ibc-token-contract = ibc3token333
ibc-chain-contract = ibc3chain333
ibc-relay-name = ibc3relay333
ibc-relay-private-key = <the pulic key of ibc3relay333>=KEY:<the private key of ibc3relay333>

ibc-sidechain-id = <peer chain id>
ibc-peer-private-key = <ibc peer public key>=KEY:<ibc peer private key>

# ------------------- p2p peers -------------------#
p2p-peer-address = <Kylin testnet p2p endpoint>
```

- configure relay node of BOS testnet  
``` 
max-transaction-time = 1000
contracts-console = true

# ------------ ibc related configures ------------ #
plugin = eosio::ibc::ibc_plugin

ibc-listen-endpoint = 0.0.0.0:5678
ibc-peer-address = 192.168.1.101:5678 
ibc-max-nodes-per-host = 5

ibc-token-contract = ibc3token333
ibc-chain-contract = ibc3chain333
ibc-relay-name = ibc3relay333
ibc-relay-private-key = <the pulic key of ibc3relay333>=KEY:<the private key of ibc3relay333>

ibc-sidechain-id = <peer chain id>
ibc-peer-private-key = <ibc peer public key>=KEY:<ibc peer private key>

# ------------------- p2p peers -------------------#
p2p-peer-address = <BOS testnet p2p endpoint>
```








