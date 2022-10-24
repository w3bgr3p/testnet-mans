# First of all stop bifrost-node

    docker stop bifrost
    docker rm bifrost

# Download bifrost-node:v1.0.1

    docker pull thebifrost/bifrost-node:v1.0.1

# Change bifrost-node version on run command

    docker run -d --network host -v "/var/lib/bifrost-data:/data" --name "bifrost" thebifrost/bifrost-node:v1.0.1 \
      --base-path /data \
      --chain /specs/bifrost-testnet.json \
      --port 30333 \
      --validator \
      --state-cache-size 0 \
      --runtime-cache-size 64 \
      --name "YOUR_CONTROLLER_ADDRESS"
