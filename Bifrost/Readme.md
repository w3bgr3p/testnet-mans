# BiFrost
## Docker
Install docker

    sudo -i
    cd $HOME
    # следующие две команды могут выдать ошибку и это нормально
    apt update && apt install curl -y && apt purge docker docker-engine docker.io containerd docker-compose -y
    rm /usr/bin/docker-compose /usr/local/bin/docker-compose
    curl -fsSL https://get.docker.com -o get-docker.sh && sh get-docker.sh
    systemctl restart docker
    curl -SL https://github.com/docker/compose/releases/download/v2.5.0/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose
    chmod +x /usr/local/bin/docker-compose
    ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
Set node name

    NODENAME=<put your node name here>
Set variable
```
echo "export NODENAME=$NODENAME" >> $HOME/.bash_profile
source $HOME/.bash_profile
```
Create dir

    mkdir -p /var/lib/bifrost-data
Set the ownership

    sudo chown -R $(id -u):$(id -g) /var/lib/bifrost-data
Run the validator node

    docker run -d -p 30333:30333 -p 9933:9933 -v "/var/lib/bifrost-data:/data" --name "$NODENAME" thebifrost/bifrost-node:latest \ 
    --base-path /data \ --chain /specs/bifrost-testnet.json \ 
    --port 30333 \ 
    --validator \ 
    --state-cache-size 0 \ 
    --runtime-cache-size 64 \ 
    --telemetry-url "wss://telemetry-connector.testnet.thebifrost.io/submit 0" \ 
    --name "$NODENAME"


---

## Useful commands
Logs

    docker logs -f $NODENAME
