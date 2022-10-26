# BiFrost for docker users

[Week 2 update](https://github.com/chaingon/testnet-mans/blob/main/Bifrost/week2.md) 

### Install docker

    sudo -i
    cd $HOME
    # you could see error it is ok proceed
    apt update && apt install curl -y && apt purge docker docker-engine docker.io containerd docker-compose -y
    rm /usr/bin/docker-compose /usr/local/bin/docker-compose
    curl -fsSL https://get.docker.com -o get-docker.sh && sh get-docker.sh
    systemctl restart docker
    curl -SL https://github.com/docker/compose/releases/download/v2.5.0/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose
    chmod +x /usr/local/bin/docker-compose
    ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose


### Create dir

    mkdir -p /var/lib/bifrost-data
### Set the ownership

    sudo chown -R $(id -u):$(id -g) /var/lib/bifrost-data
### Run the validator node (change YOUR CONTROLLER ADDRESS to your)

      docker run -d -p 30333:30333 -p 9933:9933 -v "/var/lib/bifrost-data:/data" --name "bifrost" thebifrost/bifrost-node:latest \
	--base-path /data \
    --chain /specs/bifrost-testnet.json \
    --port 30333 \
     --validator \
     --state-cache-size 0 \
    --runtime-cache-size 64 \
	--telemetry-url "wss://telemetry-connector.testnet.thebifrost.io/submit 0" \
    --name "YOUR CONTROLLER ADDRESS"

###  Logs
    docker logs -f bifrost

use it to watch for sync progress when sync will be done you can proceed
 step 1. access your docker container and install required packages

    docker exec -it bifrost /bin/bash
    cd tools
    npm install
    npm install -g npm@8.19.2
    

Now you will need your private keys which you was registered in form:
If you extracting it from metamask then you have to put 0x in start of sting
*example* 

> if extracted privat key looks like this
> `f8714217fd1d7bb49dd55a30d7ed144d817c1a3a397bd2dd3fd85f2cd0c7ca6c` then
> you have to edit it to 
> `0xf8714217fd1d7bb49dd55a30d7ed144d817c1a3a397bd2dd3fd85f2cd0c7ca6c`

Set session keys

    npm run set_session_keys -- \
    --controllerPrivate "YOUR_CONTROLLER_PRIVATE_KEY"

 **Bonding**
 for a basic node

    npm run join_validators -- \
      --controllerPrivate "YOUR_CONTROLLER_PRIVATE_KEY" \
      --stashPrivate "YOUR_STASH_PRIVATE_KEY" \
      --bond "50000"

 for a full node

    npm run join_validators -- \
      --controllerPrivate "YOUR_CONTROLLER_PRIVATE_KEY" \
      --stashPrivate "YOUR_STASH_PRIVATE_KEY" \
      --relayerPrivate "YOUR_RELAYER_PRIVATE_KEY" \
      --bond "100000"
      
  usefull commands
  
  restart
  
    docker restart bifrost
  logs
  
    docker logs -f bifrost
  delete node
  
    docker stop bifrost &&  sudo  docker  rm bifrost
    rm -rf /var/lib/bifrost-data
    
  If your valodator have status IDLE go [here](https://polkadot.js.org/apps/?rpc=wss://public-01.testnet.thebifrost.io/ws#/extrinsics) chose `bfcStaking` and `goOnline()`
Submit tx
   
