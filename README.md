## Node System Requirements

https://docs.allora.network/datasci/requirements

## Need Some Requirements for PC Users

1. Install Docker - https://www.docker.com/products/docker-desktop/

2. Install WSL - https://learn.microsoft.com/en-us/windows/wsl/install#install-wsl-command

## For Mobile Users

1. GitPod - https://www.gitpod.io/
2. Repo - https://github.com/allora-network/allora-chain


1️⃣. Install Packages
```
sudo apt update & sudo apt upgrade -y

sudo apt install ca-certificates zlib1g-dev libncurses5-dev libgdbm-dev libnss3-dev curl git wget make jq build-essential pkg-config lsb-release libssl-dev libreadline-dev libffi-dev gcc screen unzip lz4 -y
```

2️⃣ Install Python3
```
sudo apt install python3
```
```
python3 --version
```
```
sudo apt install python3-pip
```
```
pip3 --version
```

3️⃣ Install Docker
```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```
```
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
```
sudo apt-get update
```
```
sudo apt-get install docker-ce docker-ce-cli containerd.io
```
```
docker version
```

4️⃣ Install Docker-Compose
```
VER=$(curl -s https://api.github.com/repos/docker/compose/releases/latest | grep tag_name | cut -d '"' -f 4)
```
```
sudo curl -L "https://github.com/docker/compose/releases/download/$VER/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```
```
sudo chmod +x /usr/local/bin/docker-compose
```
```
docker-compose --version
```

5️⃣ Docker Permission to user
```
sudo groupadd docker
```
```
sudo usermod -aG docker $USER
```

6️⃣ Install Go
```
sudo rm -rf /usr/local/go
```
```
curl -L https://go.dev/dl/go1.22.4.linux-amd64.tar.gz | sudo tar -xzf - -C /usr/local
```
```
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bash_profile
echo 'export PATH=$PATH:$(go env GOPATH)/bin' >> $HOME/.bash_profile
```
```
source .bash_profile
```
```
go version
```

7️⃣ Install Allorad Wallet
```
git clone https://github.com/allora-network/allora-chain.git
```
```
cd allora-chain && make all
```
```
allorad version
```

8️⃣ Create a new wallet
```
allorad keys add testkey
```

9️⃣ Import & Take Faucet

➡Import your key on keplr then copy your allora address 
➡Add chain - https://tinyurl.com/3xx87w73
➡Claim Faucet - https://faucet.edgenet.allora.network/

📌Join Allora Phase 2 Points program(Connect ur Kelpr Wallet) — https://app.allora.network/points/overview

1️⃣0️⃣ Install Worker

10.1 Install
```
cd $HOME && git clone https://github.com/allora-network/basic-coin-prediction-node
```
```
cd basic-coin-prediction-node
```
```
mkdir worker-data
```
```
mkdir head-data
```
Check If Directory is Created or Not — `` ls ``

10.2 Give Certain Permissions
```
sudo chmod -R 777 worker-data
```
```
sudo chmod -R 777 head-data
```

10.3 Create head keys
```
sudo docker run -it --entrypoint=bash -v ./head-data:/data alloranetwork/allora-inference-base:latest -c "mkdir -p /data/keys && (cd /data/keys && allora-keys)"
```

10.4 Create worker keys
```
sudo docker run -it --entrypoint=bash -v ./worker-data:/data alloranetwork/allora-inference-base:latest -c "mkdir -p /data/keys && (cd /data/keys && allora-keys)"
```

1️⃣1️⃣ Copy the head-id
``` cat head-data/keys/identity ```  starts with 12D (This Is Your Head Id)

1️⃣2️⃣ Connect to Allora Chain

12.1 Delete and create new docker-compose.yml file
```
rm -rf docker-compose.yml && nano docker-compose.yml
```

12.2 Copy & Paste the following code in it
Replace head-id & WALLET_SEED_PHRASE
```
version: '3'

services:
  inference:
    container_name: inference-basic-eth-pred
    build:
      context: .
    command: python -u /app/app.py
    ports:
      - "8000:8000"
    networks:
      eth-model-local:
        aliases:
          - inference
        ipv4_address: 172.22.0.4
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/inference/ETH"]
      interval: 10s
      timeout: 5s
      retries: 12
    volumes:
      - ./inference-data:/app/data

  updater:
    container_name: updater-basic-eth-pred
    build: .
    environment:
      - INFERENCE_API_ADDRESS=http://inference:8000
    command: >
      sh -c "
      while true; do
        python -u /app/update_app.py;
        sleep 24h;
      done
      "
    depends_on:
      inference:
        condition: service_healthy
    networks:
      eth-model-local:
        aliases:
          - updater
        ipv4_address: 172.22.0.5

  worker:
    container_name: worker-basic-eth-pred
    environment:
      - INFERENCE_API_ADDRESS=http://inference:8000
      - HOME=/data
    build:
      context: .
      dockerfile: Dockerfile_b7s
    entrypoint:
      - "/bin/bash"
      - "-c"
      - |
        if [ ! -f /data/keys/priv.bin ]; then
          echo "Generating new private keys..."
          mkdir -p /data/keys
          cd /data/keys
          allora-keys
        fi
        # Change boot-nodes below to the key advertised by your head
        allora-node --role=worker --peer-db=/data/peerdb --function-db=/data/function-db \
          --runtime-path=/app/runtime --runtime-cli=bls-runtime --workspace=/data/workspace \
          --private-key=/data/keys/priv.bin --log-level=debug --port=9011 \
          --boot-nodes=/ip4/172.22.0.100/tcp/9010/p2p/'head-id' \
          --topic=1 \
          --allora-chain-key-name=testkey \
          --allora-chain-restore-mnemonic='WALLET_SEED_PHRASE' \
          --allora-node-rpc-address=https://allora-rpc.edgenet.allora.network/ \
          --allora-chain-topic-id=1
    volumes:
      - ./worker-data:/data
    working_dir: /data
    depends_on:
      - inference
      - head
    networks:
      eth-model-local:
        aliases:
          - worker
        ipv4_address: 172.22.0.10

  head:
    container_name: head-basic-eth-pred
    image: alloranetwork/allora-inference-base-head:latest
    environment:
      - HOME=/data
    entrypoint:
      - "/bin/bash"
      - "-c"
      - |
        if [ ! -f /data/keys/priv.bin ]; then
          echo "Generating new private keys..."
          mkdir -p /data/keys
          cd /data/keys
          allora-keys
        fi
        allora-node --role=head --peer-db=/data/peerdb --function-db=/data/function-db  \
          --runtime-path=/app/runtime --runtime-cli=bls-runtime --workspace=/data/workspace \
          --private-key=/data/keys/priv.bin --log-level=debug --port=9010 --rest-api=:6000
    ports:
      - "6000:6000"
    volumes:
      - ./head-data:/data
    working_dir: /data
    networks:
      eth-model-local:
        aliases:
          - head
        ipv4_address: 172.22.0.100


networks:
  eth-model-local:
    driver: bridge
    ipam:
      config:
        - subnet: 172.22.0.0/24

volumes:
  inference-data:
  worker-data:
  head-data:
```

To save: CTRL+X Then Enter Y Then Enter

1️⃣3️⃣ Run worker
```
sudo docker compose build
```
```
sudo docker compose up -d
```

1️⃣4️⃣Check your node status & Copy Worker Container ID
```
docker ps
```

1️⃣5️⃣ Replace CONTAINER_ID with the id of your docker containers

Before That Make Sure you have Claim Faucet
```
docker logs -f CONTAINER_ID
```

Success: register node Tx Hash:= XXXXXX ( Copy and save)


1️⃣6️⃣Now You Can Check Your Status By Running This Command
```
cd basic-coin-prediction-node
```
```
docker ps
```

🔶For Next Day Run This Command

#1 Open docker 1st
#2 ``` cd basic-coin-prediction-node ```
#3 ``` sudo docker compose up -d ```
#4 ``` docker ps ```
