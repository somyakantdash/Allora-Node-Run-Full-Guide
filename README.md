<h1> Node System Requirements <h1>

https://docs.allora.network/datasci/requirements

1️⃣. Install Packages

shell

sudo apt update & sudo apt upgrade -y

sudo apt install ca-certificates zlib1g-dev libncurses5-dev libgdbm-dev libnss3-dev curl git wget make jq build-essential pkg-config lsb-release libssl-dev libreadline-dev libffi-dev gcc screen unzip lz4 -y

2️⃣ Install Python3

shell

sudo apt install python3

shell

python3 --version

shell

sudo apt install python3-pip

shell

pip3 --version
3️⃣ Install Docker

shell

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

shell

echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

shell

sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
docker version

4️⃣ Install Docker-Compose

shell

VER=$(curl -s https://api.github.com/repos/docker/compose/releases/latest | grep tag_name | cut -d '"' -f 4)

shell

sudo curl -L "https://github.com/docker/compose/releases/download/$VER/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

shell

docker-compose --version

5️⃣ Docker Permission to user

shell

sudo groupadd docker

shell

sudo usermod -aG docker $USER

6️⃣ Install Go

shell

sudo rm -rf /usr/local/go

shell

curl -L https://go.dev/dl/go1.22.4.linux-amd64.tar.gz | sudo tar -xzf - -C /usr/local
shell

echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bash_profile
echo 'export PATH=$PATH:$(go env GOPATH)/bin' >> $HOME/.bash_profile

shell

source .bash_profile

go version

7️⃣ Install Allorad Wallet

shell

git clone https://github.com/allora-network/allora-chain.git

shell

cd allora-chain && make all

allorad version

8️⃣ Create a new wallet

allorad keys add testkey

9️⃣ Import & Take Faucet

➡️ Import your key on keplr then copy your allora address 
➡️ Add chain - https://tinyurl.com/3xx87w73
➡️ Faucet - https://faucet.edgenet.allora.network/

📌 Join Allora Phase 2 Points program — Click Here

1️⃣🔤 Install Worker

10.1 Install

shell

cd $HOME && git clone https://github.com/allora-network/basic-coin-prediction-node

shell

cd basic-coin-prediction-node

shell

mkdir worker-data

shell

mkdir head-data

Check If Directory is Created or Not — ls

10.2 Give Certain Permissions

shell

sudo chmod -R 777 worker-data

shell

sudo chmod -R 777 head-data

10.3 Create head keys

shell

sudo docker run -it --entrypoint=bash -v ./head-data:/data alloranetwork/allora-inference-base:latest -c "mkdir -p /data/keys && (cd /data/keys && allora-keys)"

10.4 Create worker keys

shell

sudo docker run -it --entrypoint=bash -v ./worker-data:/data alloranetwork/allora-inference-base:latest -c "mkdir -p /data/keys && (cd /data/keys && allora-keys)"

1️⃣1️⃣ Copy the head-id

cat head-data/keys/identity - starts with 12D (This Is Your Head Id)

1️⃣2️⃣ Connect to Allora Chain

12.1 Delete and create new docker-compose.yml file

shell

rm -rf docker-compose.yml && nano docker-compose.yml
