### Ставим докер

```
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
```

### Создаем каталог

```
mkdir -p /var/lib/bifrost-data
```

### Задаем права

```
sudo chown -R $(id -u):$(id -g) /var/lib/bifrost-data
```

### Запускаем ноду (меняем YOUR CONTROLLER ADDRESS HERE на свой аддресс)

```
  docker run -d -p 30333:30333 -p 9933:9933 -v "/var/lib/bifrost-data:/data" --name "bifrost" thebifrost/bifrost-node:latest \
--base-path /data \
--chain /specs/bifrost-testnet.json \
--port 30333 \
 --validator \
 --state-cache-size 0 \
--runtime-cache-size 64 \
--telemetry-url "wss://telemetry-connector.testnet.thebifrost.io/submit 0" \
--name "YOUR CONTROLLER ADDRESS"
```

### Логи

```
docker logs -f bifrost
```

запускаем логи ждем пока нода отсинхронизируется, только потом едем дальше

### Переходим в наш докер контейнер

```
docker exec -it bifrost /bin/bash
cd tools
npm install
```

Теперь нам нужны приват ключи от кошельков которые мы регали в форму, как дергать ключи из метамаска надеюсь все в курсе. Ключи для последующей работы нужно будет чутка подредактировать добавив 0x в начале

_пример_

> если то что вы извлекли из метамаск выглядит так `f8714217fd1d7bb49dd55a30d7ed144d817c1a3a397bd2dd3fd85f2cd0c7ca6c` то вам нужно сделать чтобы оно выглядело вот так `0xf8714217fd1d7bb49dd55a30d7ed144d817c1a3a397bd2dd3fd85f2cd0c7ca6c`

### Задаем ключи сессии

```
npm run set_session_keys -- \
  --controllerPrivate "YOUR_CONTROLLER_PRIVATE_KEY"
```

### Делаем стейк в нашего валика

если у вас базовая нода так

```
npm run join_validators -- \
  --controllerPrivate "YOUR_CONTROLLER_PRIVATE_KEY" \
  --stashPrivate "YOUR_STASH_PRIVATE_KEY" \
  --bond "50000"
```

или если фулл нода то так

```
npm run join_validators -- \
  --controllerPrivate "YOUR_CONTROLLER_PRIVATE_KEY" \
  --stashPrivate "YOUR_STASH_PRIVATE_KEY" \
  --relayerPrivate "YOUR_RELAYER_PRIVATE_KEY" \
  --bond "100000"
```

