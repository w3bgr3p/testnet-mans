# Validator-setup & post installation management


Load variables into system
```
source $HOME/.bash_profile
```

Next you have to make sure your validator is syncing blocks. You can use command below to check synchronization status
```
hid-noded status 2>&1 | jq .SyncInfo
```

### Create wallet
To create new wallet you can use command below. Don’t forget to save the mnemonic
```
hid-noded keys add $WALLET
```

(OPTIONAL) To recover your wallet using seed phrase
```
hid-noded keys add $WALLET --recover
```

To get current list of wallets
```
hid-noded keys list
```

### Save wallet info
Add wallet and valoper address into variables 
```
HYPERSIGN_WALLET_ADDRESS=$(hid-noded keys show $WALLET -a)
HYPERSIGN_VALOPER_ADDRESS=$(hid-noded keys show $WALLET --bech val -a)
echo 'export HYPERSIGN_WALLET_ADDRESS='${HYPERSIGN_WALLET_ADDRESS} >> $HOME/.bash_profile
echo 'export HYPERSIGN_VALOPER_ADDRESS='${HYPERSIGN_VALOPER_ADDRESS} >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### Fund your wallet
In order to create validator first you need to fund your wallet with testnet tokens.
```
N/A
```

### Create validator
Before creating validator please make sure that you have at least 1 strd (1 strd is equal to 1000000 uhid) and your node is synchronized

To check your wallet balance:
```
hid-noded query bank balances $HYPERSIGN_WALLET_ADDRESS
```
> If your wallet does not show any balance than probably your node is still syncing. Please wait until it finish to synchronize and then continue 

To create your validator run command below
```
hid-noded tx staking create-validator \
  --amount 100000000uhid \
  --from $WALLET \
  --commission-max-change-rate "0.01" \
  --commission-max-rate "0.2" \
  --commission-rate "0.1337" \
  --min-self-delegation "1" \
  --pubkey  $(hid-noded tendermint show-validator) \
  --moniker $NODENAME \
  --chain-id $HYPERSIGN_CHAIN_ID
```

## Security
To protect you keys please make sure you follow basic security rules

### Set up ssh keys for authentication
Good tutorial on how to set up ssh keys for authentication to your server can be found [here](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-on-ubuntu-20-04)

### Basic Firewall security
Start by checking the status of ufw.
```
sudo ufw status
```

Sets the default to allow outgoing connections, deny all incoming except ssh and 26656. Limit SSH login attempts
```
sudo ufw default allow outgoing
sudo ufw default deny incoming
sudo ufw allow ssh/tcp
sudo ufw limit ssh/tcp
sudo ufw allow ${HYPERSIGN_PORT}656,${HYPERSIGN_PORT}660/tcp
sudo ufw enable
```
## Usefull commands

### Check your validator key
```
[[ $(hid-noded q staking validator $HYPERSIGN_VALOPER_ADDRESS -oj | jq -r .consensus_pubkey.key) = $(hid-noded status | jq -r .ValidatorInfo.PubKey.value) ]] && echo -e "\n\e[1m\e[32mTrue\e[0m\n" || echo -e "\n\e[1m\e[31mFalse\e[0m\n"
```

### Get list of validators
```
hid-noded q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```

## Get currently connected peer list with ids
```
curl -sS http://localhost:${HYPERSIGN_PORT}657/net_info | jq -r '.result.peers[] | "\(.node_info.id)@\(.remote_ip):\(.node_info.listen_addr)"' | awk -F ':' '{print $1":"$(NF)}'
```


### Service management
Check logs
```
journalctl -fu hid-noded -o cat
```

Start service
```
sudo systemctl start hid-noded
```

Stop service
```
sudo systemctl stop hid-noded
```

Restart service
```
sudo systemctl restart hid-noded
```

### Node info
Synchronization info
```
hid-noded status 2>&1 | jq .SyncInfo
```

Validator info
```
hid-noded status 2>&1 | jq .ValidatorInfo
```

Node info
```
hid-noded status 2>&1 | jq .NodeInfo
```

Show node id
```
hid-noded tendermint show-node-id
```

### Wallet operations
List of wallets
```
hid-noded keys list
```

Recover wallet
```
hid-noded keys add $WALLET --recover
```

Delete wallet
```
hid-noded keys delete $WALLET
```

Get wallet balance
```
hid-noded query bank balances $HYPERSIGN_WALLET_ADDRESS
```

Transfer funds
```
hid-noded tx bank send $HYPERSIGN_WALLET_ADDRESS <TO_HYPERSIGN_WALLET_ADDRESS> 10000000uhid
```

### Voting
```
hid-noded tx gov vote 1 yes --from $WALLET --chain-id=$HYPERSIGN_CHAIN_ID
```

### Staking, Delegation and Rewards
Delegate stake
```
hid-noded tx staking delegate $HYPERSIGN_VALOPER_ADDRESS 10000000uhid --from=$WALLET --chain-id=$HYPERSIGN_CHAIN_ID --gas=auto
```

Redelegate stake from validator to another validator
```
hid-noded tx staking redelegate <srcValidatorAddress> <destValidatorAddress> 10000000uhid --from=$WALLET --chain-id=$HYPERSIGN_CHAIN_ID --gas=auto
```

Withdraw all rewards
```
hid-noded tx distribution withdraw-all-rewards --from=$WALLET --chain-id=$HYPERSIGN_CHAIN_ID --gas=auto
```

Withdraw rewards with commision
```
hid-noded tx distribution withdraw-rewards $HYPERSIGN_VALOPER_ADDRESS --from=$WALLET --commission --chain-id=$HYPERSIGN_CHAIN_ID
```

### Validator management
Edit validator
```
hid-noded tx staking edit-validator \
  --moniker=$NODENAME \
  --identity=<your_keybase_id> \
  --website="<your_website>" \
  --details="<your_validator_description>" \
  --chain-id=$HYPERSIGN_CHAIN_ID \
  --from=$WALLET
```

Unjail validator
```
hid-noded tx slashing unjail \
  --broadcast-mode=block \
  --from=$WALLET \
  --chain-id=$HYPERSIGN_CHAIN_ID \
  --gas=auto
```

### Delete node
This commands will completely remove node from server. Use at your own risk!
```
sudo systemctl stop hid-noded
sudo systemctl disable hid-noded
sudo rm /etc/systemd/system/hypersign* -rf
sudo rm $(which hid-noded) -rf
sudo rm $HOME/.hid-node* -rf
sudo rm $HOME/hid-node -rf
sed -i '/HYPERSIGN_/d' ~/.bash_profile
```

