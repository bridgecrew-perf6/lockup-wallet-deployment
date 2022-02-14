## Deploy and run lockup Contract end to end 
This flow covers the deploy and execution of [uni-lockup-wallet](https://github.com/ton-blockchain/lockup-wallet-contract/blob/main/universal/uni-lockup-wallet.fc)

official frontend for creating the locking up transaction  [repository](https://github.com/toncenter/lockup-sender)

### compile [uni-lockup-wallet](https://github.com/ton-blockchain/lockup-wallet-contract/blob/main/universal/uni-lockup-wallet.fc) 
`func -o uni-lockup-wallet-code.fif -SPA stdlib.fc uni-lockup-wallet.fc` -> uni-lockup-wallet-code.fif


### build lockup wallet
- deploy **uni-lockup-wallet** contract  and generate 2 new keys (pk files) , 1. owner 2. config  using **deploy.fif**  to contract address will be referred as **<lockup-wallet>**

`fift -s src/deploy.fif <workchain> <subwallet_id>    <owner_lockup_pk> <config_pk` result `<lockup_address` and `<lockup-query.boc`
### deploy lockup wallet
` lite-client -C global.config.json -c 'sendfile  src/lockup-query.boc`


## deploy v3 wallet using **config.pk** (we will refer it as config-wallet)
`fift -s new-wallet-v3.fif <workchain> <subwallet_id> <config_pk`   >output BOC file -> `config.pk-query.boc`

- 3.send a new message from **config-wallet** to lockup wallet , the message is signed by **config.pk**, and it has TON to lock, unlock timestamp, and if restricted flag (or locked funds can be used for nominators)
`fift -s query-new-lockup.fif <time_to_lock_sec> <allow_funds_staking> <pk_flie`  

`fift -s query-new-lockup.fif 300 0 config` BOC output: `conifg-rs-transfer.boc` 

#### get seqno for lockup-wallet (<lockup-wallet> in stage 1) 
` lite-client -C global.config.json -c 'runmethod <lockup-wallet> seqno'`  -> result `<lockup-seqno`
> build wallet query (wrap lockup query)
`fift -s wallet-v3-query-builder.fif  <config.pk> <lockup-wallet>  <subwallet_id> <lockup-seqno> <amount_of_ton_to_lock> -B restricted_transfer.boc lockup-transfer`

`lite-client -C global.config.json -c 'sendfile src/lockup-transfer'`

> send this tx from config-wallet to lockup-wallet, so we use wallet-v3-query-builder ``  
- [ ] 4. owner.pk wallet should withdraw the remaining balance
`fift -s wallet-v3-query-builder.fif  <config.pk> <lockup-wallet> <subwallet_id> <lockup-seqno> <amount_of_ton_to_lock> -B restricted_transfer.boc lockup-transfer`

### Using a Wallet App with the contract 
- [ ] force TON web wallet to use owner.pk ( this probably would need code modification to force another private key)
- [ ] Or the better scenario is to to export the 24 words mnemonic to pk file, and deploy the lockup contract with the mnemonic base private key. after that modify ton web wallet target address (localstorage change) , and everything should work [ton-crypto](https://github.com/tonwhales/ton-crypto/blob/59b8b9b1a1bce1407c55a38a39115ed7aaaf333b/src/index.ts).


# get balance at , so we can check in each timestamp the funds status , unlocked, locked, and restricted 

lite-client -C global.config.json -c 'runmethod <> get_balances_at <unix_timestamp>' -> result [ <unlocked>, <restricted>, <locked>]


## withdraw money from wallet

1. get seqno from lockup-wallet 
` lite-client -C global.config.json -c 'runmethod <lockup_address> seqno'` <seqno>

`  fift -s wallet-v3-query-builder.fif  owner <lockup_address> <subwallet_id> <seqno> <ton_amount>   lockup-withdraw`
### sendfile via lite-client 
`lite-client -C global.config.json -c 'sendfile src/lockup-withdraw.boc'`


