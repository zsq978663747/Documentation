EOSIO IBC Priciple and Design
---------
*Author:Simon [Github](https://github.com/vonhenry)*

This paper introduces the technical principle of IBC, the contracts and ibc_plugin developed by boscore team.

In order to realize the inter-blockchain transfer of token between two EOSIO blockchains, first, we need to 
solve two problems: 1. How to realize the lightweight client, 2. How to ensure the integrity and reliability 
of inter-blockchain transactions, how to prevent double spend and replay attacks.

The EOS mainnet and BOS mainnet are illustrated below. However, this document applicable for
any two EOSIO architecture blockchains.


### 1. Terminology
- BOSIBC  
  IBC contracts and ibc_plugin developed by the BOSCORE technical team.
  
### 2. Key concepts and data structures
- Simple Payment Verification (SPV)  
  Simple Payment Verification was first proposed in Bitcoin White Paper of Satoshi (https://bitcoin.org/bitcoin.pdf) 
  to verify that a transaction exists in a blockchain.`SPV client` stores contiguous block headers, but no blocks' body,
  so it only takes up a small amount of storage space. When receive a transaction and the `merkle path` of the transaction,
  `SPV client` can used to verify whether this transaction exist in the blockchain or not.

- Lightwight client (lwc)  
  also called `SPV client', is a lightweight chain composed of block headers.

- Merkle path  
  In order to verify whether a transaction exists in a blockchain, only the original data of the transaction 
  and the transaction's `merkle path` is needed, instead of the whole block. then calculate the merkle root and
  compaire with the merkle root of block header, if equal, indicates that the transaction exists in that blockchain.
  `merkle path` also known as `merkle branch`.

- Block Producer Schedule  
  The `BP Schedule` is a EOSIO architecture blockchain technology, which is used to determine the rights that which 
  producer can produce blocks. The new version `BP Schedule` comes into effect after passing the validation of
  the last batch of `BP Schedule`. In order to ensure strict handover of BP rights, it is a core technology of 
  IBC system logic, that the lightweight client must follow the corresponding `BP Schedule` with it's mainnet.

- forkdb  
  When the EOSIO node runs, there are two underlying DBs used to store block information. 
  One is `blog`, i.e. `block log`, which is used to store irreversible blocks, and the other is `forkdb`, 
  which is used to store reversible blocks. Forkdb stores part of the block information at the top of the 
  current blockchain. A block must be accepted by forkdb before it finally enters the irreversible block. 
  The lightweight client mainly refers to the logic of forkdb.
  
### 3. Lightweight Client

In order to solve the cross-chain problem, the first problem to be solved is how to realize the lightweight client.
1. At where should the lightweight client runs? In the contract or out of the contract, for example, in plugin layer?
2. If running in contract, is it to synchronize all block header data of the opposite blockchain in real time, 
   or to synchronize part of block header data to verify transactions according to need, because synchronizing
   all block header data will consume a lot of CPU resources of two chains.
3. If running in contract, how to ensure the credibility of the lightweight client, how to prevent malicious attacks, 
   how to achieve complete decentralization, does not depend on the trust of any relay node.


#### 3.1 Should the Lightweight Client Run in the Contract
Bitcoin's light client was originally run on a single node (such as a personal computer or a decentralized 
Bitcoin mobile wallet) to verify the existence of transactions and to see the depth of that block. Specific 
implementation can be referred to [breadwallet](https://github.com/breadwallet/breadwallet-core).

IBC and Decentralized Wallet have different requirements for light clients. Decentralized wallets generally run
on personal mobile apps, providing transaction verification services for individual users, while IBC systems need 
light clients that are open, accessible and trustworthy to everyone. From this point of view, a light client that
can gain public trust can only run in the contract, because only the contract data is globally consistent and 
can not be tampered with. It is impossible to implement a trusted light client outside the contract, so BOSIBC 
run the light client in the contract. See source code of [ibc.chain](https://github.com/boscore/ibc_contracts/tree/master/ibc.chain).

#### 3.2 Whether to synchronize all block headers
In the light client of Bitcoin and ETH, a starting block and some subsequent verification block will be provided, 
and the light client will synchronize the full block headers after that starting block. Bitcoin produces only 4 Mb 
of block headers per year. According to the current storage capacity of mobile devices, it can be fully accommodated, 
and synchronization of these blocks will not consume a lot of computing resources of mobile devices. However, 
the situation of EOSIO is quite different. EOSIO contract consumes 0.5 milliseconds of CPU to add a block header to the contract 
and 0.2 milliseconds to deleted a block header, so 0.7 ms of CPU is needed for every block header procession.
Assuming that we want to synchronize all the block headers of the other EOSIO chain, according to the total CPU 
time of each block, now is 200ms, that is to say, `0.7ms/200ms = 0.35%` of the whole chain is needed to achieve 
real-time full synchronization. According to the actual total amount of delgated token of the whole network is about 
400 million, in order to make the IBC system works when the CPU is busy, the account which push block headers and ibc 
transactions needs to be delgate `400 million * 0.35% = 1.4 million` token, which is a large amount. Because EOSIO 
advocates multi side-chain ecosystem, let's assumed that there will be multiple side chains and EOSIO main network
to achieve cross-chain in the future, and between side chains and side chains, and it also will realizes one-to-many
cross-chains operation, assuming the one-to-ten situation, each chain needs to maintain 10 light clients, 
in order to maintain these light clients, it needs to consume 3.5% of the whole network CPU of a single chain,
this proportion is too high, therefore, we need to find a more reasonable solution.

The process of designing inter-blockchain communication is a process of finding credible evidence. 
Is there a scheme that does not need to synchronize the whole block information, but also can guarantee the credibility
of light clients? The bottom of EOSIO source code has been prepared for this purpose.
Let's assume that if BP schedule never change, then at any time, when the ibc.chain contract obtains a series of
block headers passed by signature verification, such as `n ~ n+336`, and more than two-thirds of the active BP
is producing blocks, we can be sure that the `n`th block is irreversible, and can be used to verify cross-chain transactions.
Later, we need to consider the situation of BP schedule replacement. When BP schedule replacement occurs, 
we will not accept transaction verification until the replacement is completed. The relatively complex process
of BP replacement is dealt with, which will be described in more detail below.
Therefore, using this scheme can greatly reduce the amount of block headers that need to be synchronized,
and only when the BP list is updated or cross-chain transactions occured, that it's needed synchronized block headers.

In order to achieve this goal, the concept of `section` is introduced in ibc.chain. 
A section records a batch of continuous block headers. Section does not store block headers, instead, 
it records the first block number (first) and the last block number (last) of the block header, block headers are
stored in `chaindb`. Each section has a valid bool value. When there is no BP schedule replacement,
as long as two-thirds of the active BP is producing blocks and `last - first > lib_depth`, then 
the block of `first to last - lib_depth` is considered irreversible and can be used to verify cross-chain transactions.
When BP schedule is replaced, the 'valid' of that section becomes false and no transaction validation is accepted 
until schedule replacement is finished and 'valid' becomes true, then continue cross-chain transaction validation.


#### 3.3 How to Ensure the Reliability of Light Clients
**3.3.1 forkdb**  
*1. How to append a new block to forkdb*  
A running eosio node maintains two underlying data structures 
[blog](https://github.com/EOSIO/eos/blob/master/libraries/chain/include/eosio/chain/block_log.hpp)
and [forkdb](https://github.com/EOSIO/eos/blob/master/libraries/chain/include/eosio/chain/fork_database.hpp),
blog is used to store irreversible block information, its stored data is serialized `signed_block`, 
forkdb is used to store reversible block information, and its stored data is `block_state`.
`block_state` contains more block-related information than `signed_block`. A block must first be appended to forkdb 
before it can eventually become irreversible and remove from forkdb into blog.
How can the `block_state` information of a block be obtained? It is not thart BP of the production block transfers 
`block_state` to other BP nodes and full-nodes, P2P network eos eosio pass only 
[signed_block](https://github.com/EOSIO/eos/blob/master/plugins/net_plugin/include/eosio/net_plugin/protocol.hpp#L142),
When a node receives a `signed_block` over P2P network, it uses this `signed_block` to construct `block_state` 
then verify the BP signature. [Relevance function](https://github.com/EOSIO/eos/blob/master/libraries/chain/block_header_state.cpp#L144),
The key points should be clarified are: 1. `blockroot_merkle`, 2. `get_scheduled_producer()`, 3. `verify_signee()`.

The signature verification related function are:
```
  digest_type   block_header_state::sig_digest()const {
     auto header_bmroot = digest_type::hash( std::make_pair( header.digest(), blockroot_merkle.get_root() ) );
     return digest_type::hash( std::make_pair(header_bmroot, pending_schedule_hash) );
  }

  public_key_type block_header_state::signee()const {
    return fc::crypto::public_key( header.producer_signature, sig_digest(), true );
  }

  void block_header_state::verify_signee( const public_key_type& signee )const {
     EOS_ASSERT( block_signing_key == signee, wrong_signing_key, "block not signed by expected key",
                 ("block_signing_key", block_signing_key)( "signee", signee ) );
  }
```

The first step in verifying signature is to obtain signature digest, i.e. `sig_digest()`, 
which uses `header.digest()`, `blockroot_merkle.get_root()` and `pending_schedule_hash`.
The second step is to obtain the signature public key, i.e. `signee()`, which compute the BP public key through
`producer_signature ` and `sig_digest()`.
The third step is to verify whether the public key is correct, i.e. `verify_signee()`, which is called in 
`block_header_state:: next()`, after validation, a block is added to forkdb main branch.

So every block which added to forkdb has undergone a very rigorous and comprehensive verification.
The core logic contains `blockroot_merkle`, `get_scheduled_producer()` and `verify_signee()`.
The ibc.chain contract fully inherits the rigorous validity of forkdb.

*2. How forkdb handles forks*  
When adding a new block, which results in a different head.id for forkdb and head.id for controller_impl, the branch is re-selected.
source code refers to `eosio::chain::controller_impl::push_block()`and `eosio::chain::controller_impl::maybe_switch_forks()`;

*3. How does LIB determine*  
The consensus approach currently used by EOSIO is dpos, also called pipelined bft, which sets `required_confs` 
when constructing a block `block_header_state`, which is 2/3+1 of the current number of active BPs.
In the case of 21 BP, `required_confs` is 15. Each block has `header.confirmed`, which is used to confirm 
the previous blocks. when a block gets a confirmation,its `required_confs` will be reduced by 1. 
When `required_confs` of a block is reduced to zero, the block will be nominated `dpos_proposed_irreversible_blocknum`
by the latest block (head of forkdb). When a block obtains 2/3 BPs nominations, it becomes an irreversible block, 
In other words, it enters LIB. Since the confirmation information is passed in block header, 
so a block from generation to enter the LIB requires a total of 2 * 2/3 round, that is `12 * (14 * 2) = 336`. 
When BP is registered, 12 blocks are continuously produced each time. Only the `header.confirmed` of the first 
block is non-zero, so when a BP starts to come out of a block, only the first block will raise the LIB, 
so the actual head and LIB gap is between 325 and 336, but in the case of BP loss blocks, the head and LIB gap may be smaller than
325 to 336.

*4. How does the BP list change*  
In the blockchain of pow, such as bitcoin and ethereum, the fork with the greatest cumulative computing power 
is chosen as the main branch. Only when a block contains certain computing power can it be recognized and 
eventually become irreversible. In EOS which adope dpos as consensus algorithm, the block's recognized mark 
is BP signature, so BP list plays an important role in EOSIO. IBC's light clients also need to maintain 
the replacement of BPs lists. Therefore, it is necessary to thoroughly analyze the replacement logic of BP list.

First, in the `onblock ()` function of the system contract `eosio. system`, the system tries to update the BP list 
`update_elected_producers(timestamp)` every minute, this function eventually calls the wasm interface
`set_proposed_producers()`, after a series of checks, the new scheme and the current block number are stored 
in the `global_property_object`.

In the second part, when the block becomes irreversible, a new list of `header.new_producers` will be set in the 
current pending block, and the `global_property_object` object will be reset.
The pending block number is recorded to `pending_schedule_lib_num`, at which point a new list can be seen 
in the nodeos log; for detailed logic, please refer to `controller:: start_block() // Promote proposed schedule to pending schedule.`
That is to say, the change of the new list from `proposed schedule` to `pending schedule` takes about 325 to 336 blocks. 
From here on, the `pending_schedule.version` of the `block_header_state` block will be 1 larger than `active_schedule.version`.

Third, when `pending_schedule_lib_num` becomes irreversible, `active_schedule` will be replaced by `pending schedule`, 
and the whole BPs replacement process is completed. from `pending schedule` to `active_schedule`, 
it also takes about 325 to 336 blocks.

The light client of IBC system also needs to inherit these logic of forkdb in order to realize the trusted light client. 
However, the light client is implemented in the contract, and the characteristics and limitations of the contract 
need to be fully considered, therefore, many adjustments need to be made in the details of implementation.

**3.3.2 eosio::table("chaindb"), "forkdb" of ibc.chain contract**  

*1. How to determine the LIB of the Lightweight Client (lwc)*  
There are two schemes. One is to maintain a whole set of `confirm_count`, `confirmations` and other `block_header_state` 
related information according to the logic of forkdb, calculate LIB every time a block is added
The advantage of this method is that real-time LIB values can be obtained accurately. However, for light client, 
it concern is that the blocks are irreversible, rather than real-time LIB values. Is there a simpler solution?
According to the above logic, if more than two-thirds of BPs is active, and the depth of one block exceeds 336, 
then the block must be irreversible and can be used to verify cross-chain transactions. 
It can simplify the complexity of forkdb in the contract.

The `lib_depth` of the table `global` in the ibc.chain contract is a depth value. When the depth of a block exceeds 
this value in a continuous block headers, it is considered irreversible and can be used to verify the transaction.
This value should be set to 336. When the light client checks that more than 2/3 of BPs is active, the blocks 
with depth over 336 is considered irreversible. However, add and delete blocks in the contract
It is very CPU-consuming. The actual test shows that it takes 0.5ms CPU to add a block header and 0.2ms to 
delete a block header, so 0.7ms CPU is needed for each block header. Whether there will be a huge fluctuation in 
the mainnet, which leading to all the blocks that not entered lib to roll back? This is possible and has actually happened. 
Therefore, it is necessary to ensure that a cross-chain transaction enters lib and then start processing it.
Plugins can play a role at this time. In the ibc_plugin developed by BOSCORE technical team, parameters are set 
to deal with only a certain depth of cross-chain transactions. Thus, the depth in the ibc_plugin and the depth in 
the contract add up to more than 336 is works. This is only the initial practice of BOSIBC, the goal is to avoid 
consuming huge CPU. The next upgrade, consideration may be given to setting the depth of the contract to 336, 
thus, it does not depend on the depth of relay at all. However, adding depth value in ibc_plugin or ibc.chain
will directly prolong the arrival time of cross-chain transactions, thus affecting the user experience. 
Therefore, it is necessary to determine a reasonable value according to the actual situation, so as to ensure 
sufficient security without losing a good user experience.

*2. table chaindb*  
Table chaindb is the "forkdb" in ibc.chain contract. There is no blog structure in ibc.chain contract. 
Unlike `block_header_state`, which is used in forkdb in eosio, chaindb has done a lot of streamlining, only reserved
`Block_num, block_id, header, active_schedule, pending_schedule, blockroot_merkle, block_signing_key` 7 elements, 
because BPs schedule takes up a lot of space, so it stores actual schedule information in another table `prodsches`, 
and uses ID reference in chaindb to save memory and wasm CPU consumption.

*3. Genesis Block Header of Lightweight Client*  
The light client needs a credible starting point, which is the genesis block header of the light client. 
The verification of all the block headers behind is based on the genesis block header.
Genesis block header need `blockroot_merkle`, `active_schedule` and `header` information in `block_header_state`, 
in order to verify block signatures. The source code is `chain::chaininit()`. The most important restriction is 
that the `pending_schedule` of the genesis block header must be the same as `active_schedule`, 
because its difference means this block is a block of the process of BPs schedule replacement. 
If we use the block in the process of BPs schedule replacement, we need to synchronize the subsequent blocks, 
Until `active_schedule` is replaced by `pending_schedule`, which increased complexity, 
so such a block header is not suitable to be a genesis block header.

*4. How do lightweight client append new block headers*  
The way light clients add headers is very similar to forkdb of eosio. See ibc.chain `pushheader()` for the source code.
The first step is to verify that whether it can connect to the latest section by block number.
The second part is whether it is necessary to deal with fork and delete old branch. 
In ibc.chain, multiple branches are not saved at the same time. Instead, the former is replaced by the latter to achieve fork processing.
Third, verify whether it can connect to the latest section by block id.
The fourth part, constructs `block_header_state` and verifies BP signature
The core logic is constructing of `block_header_state`, in which the `BP schedule` replacement is processed 
to ensure that more than two-thirds of the active BPs producing block (see `section_type::add()`).

*5. Section*  
Section is the core concept and innovation of chaindb, which means a batch of continuous block headers. 
Section management is also the core logic of ibc.chain. The purpose of using section is to reduce CPU consumption, 
and only when BP schedule changes or inter-blockchain transactions occur, then it needs to synchronize a batch of  
block headers. The starting block header of any section cannot be a block in the process of BPs schedule replacement. 
That is to say, the `pending_schedule.version` of the starting block header of any section must be equal to
`active_schedule.version`, and the `active_schedule` of the starting block header of a section must be the 
same as the `active_schedule` of the previous section, This ensures that there must be no replacement of 
BPs schedules between any two sections, and that the complete process of each replacement of BP schedules must 
be completed **in** a section, this ensures the credibility of section data.


#### 4. Inter-blockchain Transactions
*1. Inter-blockchain Transaction Trilogy*  
A inter-blockchain transaction is divided into three processes. The following is an example of transfer EOS token 
from the EOS mainnet to the BOS mainnet. First, the user initiates a inter-blockchain transaction `orig_trx` 
on the EOS mainnet, the transaction information is recorded in the `origtrx` table of the `ibc.token` contract 
on the EOS mainnet side. When the transaction enters lib, IBC relay on the EOS mainnet side (relay_eos ) will 
get the transaction and transaction-related information (block information and Merkle path), then pass it to 
the relay of BOS mainnet side (relay_bos); relay_bos constructs 'cash' transaction and calls the **cash** 
action of the `ibc.token` contract on the BOS mainnet side. If the call succeeds, in the **cash** function, 
the corresponding token will be issued to the target user; when the block contain the **cash** transaction enters lib, 
relay_bos will get `cash_trx` and related information (block information and Merkle path), then passing them to 
relay_eos, relay_eos constructs the **cashconfirm** transaction and calls the **cashconfirm** action of the `ibc.token`
contract on the EOS mainnet side, which deletes the original transaction record in `ibc.token` contract.
then a complete inter-blockchain transaction has been completed.

*2. Failed Inter-blockchain Transactions*  
Inter-blockchain transactions may fail, for example, the designated account does not exist on the peer chain, 
or because of the bad network environment, the call to the cash interface fails, 
Failed inter-blockchain transactions will be rolled back, that is to say, the assets of users are returned in the original way.
However, the current IBC system is transaction-driven, and failed IBC transactions need to wait until success
IBC transactions are completed before they can be rolled back. 
(Note: Subsequent upgrades will roll back failed transactions as soon as possible)


*3. How to prevent replay attacks, i.e. double-flower attacks*  
Preventing double-flower attacks can be divided into two stages:
1. A successful cross-chain transaction can only execute **cash** once, otherwise it will result in repeated cash.
2. For each **cash** transaction, the relevant information must be sent back to the original chain to execute the 
**cashconfirm** in order to remove the original transaction information recorded in the contract,
otherwise, the token will be issued to the user on the destination chain, then the token on the original chain returned to the user.

The **cash** function is the core logic of ibc.token. The `ibc.token` contract records the original transaction ids `orig_trx_id`, 
which recently executed cash, and the block number of `orig_trx` of the new cash must be greater than or equal to 
the highest block number of all `orig_trx`, that is to say, cash must be done in the order of blocks in the original 
chain according to the original transaction. (When cash is executed, the order of cross-chain transactions 
in one same block in the original chain is arbitrarily orderable.)
Combined with trx_id checking, we can ensure that a cross-chain transaction can only execute cash once.

Similarly, the cashconfirm action checks the cash transaction number `seq_num', which must be incremented one by one 
to ensure that all cash transactions on the destination chain delete the original transaction record on the original chain.
This ensures that there will be no double spend.


### 5. ibc_plugin 
The role of ibc_plugin is divided into two parts: 1. light client synchronization; 2. inter-blockchain transactions delivery.
For core logic, see `ibc_plugin_impl::ibc_core_checker()`.
ibc_plugin refers mainly to the framework of net_plugin.


### 6. Questions and Answers
*1. Question: relay's authority is used in many actions of IBC contracts, so does the IBC system depend on trust in relays?*   
Answer: At present, for the sake of security and fast function iteration, relay privileges are added. 
With the gradual improvement and maturity of IBC, BOS IBC will support multiple relay channels to avoid single-point risk.

There are two considerations in validating relay authority: 
first. The ibc.chain contract uses the **section** mechanism, and the current logic does not allow adding blocks to the old section,
it is also not allowed to add a block header before a section. If anyone can call the "pushsection" action, 
assuming that the range of blocks that should be pushed is 1000-1300, the deliberate troublemaker may push 1100-1300 first,
as a result, 1000-1100 can not be pushed any more, leading to some cross-chain transactions can not be successful. 
(Note, this issue will be considered in future versions of optimization.) 
second, Considering that IBC system carries a large number of user assets, and the system has not passed the long-term market test, 
so the relay authority is increased to reduce security risks.


### 7. Upgrade plan
*1. Compatible with PBFT consistency algorithm*  

*2. Supporting inter-blockchain transactions between multiple side chains in a more elegant way*  

*3. Supporting inter-blockchain commuincation for more types of data than "token"*  


### 8. Reference
[Inter-blockchain Communication via Merkle Proofs with EOS.IO](https://steemit.com/eos/@dan/inter-blockchain-communication-via-merkle-proofs-with-eos-io) *Bytemaster*    
[Bitcoin](https://bitcoin.org/bitcoin.pdf) *Satoshi Nakamoto*   
[Chain Interoperability](https://static1.squarespace.com/static/55f73743e4b051cfcc0b02cf/t/5886800ecd0f68de303349b1/1485209617040/Chain+Interoperability.pdf) *Vitalik Buterin*   



