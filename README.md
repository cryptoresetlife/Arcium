🚀 Arcium 节点部署教程（适用于 Ubuntu / WSL2）

⭐ 本教程为 Arcium ARX Node 公测网的完整部署流程
⭐ 零隐私，可直接用于 GitHub 公开分享
⭐ 所有步骤均已实机验证稳定可用

📚 目录

简介

系统要求

1. 安装系统依赖

2. 安装 Node.js + Yarn

3. 安装 Rust

4. 安装 Solana CLI

5. 安装 Arcium 工具链

6. 生成 Keypair

7. 切换 Solana 到 Devnet

8. 领取 Devnet SOL

9. 初始化 ARX Node

10. 创建 node-configtoml

11. 创建 Docker Compose

12. 启动节点

13. 查看运行日志

14. 验证节点状态

FAQ: 关于 Cluster 功能

简介

Arcium 是构建在 Solana 上的去中心化多方计算（MPC）网络。
本指南将教你如何部署一个 可正常激活、可正常上链、可自动运行的 ARX 节点。

适用于：

Linux

Ubuntu 22.04

Windows WSL2

系统要求
项目	最低要求
CPU	8 核
RAM	16GB
网络	稳定、可触达公网
GPU	不需要
系统	Ubuntu 22.04 / WSL2
1. 安装系统依赖
sudo apt update && sudo apt upgrade -y

sudo apt install -y curl iptables build-essential git wget lz4 jq make gcc nano automake \
autoconf tmux htop nvme-cli pkg-config libssl-dev libleveldb-dev tar clang \
bsdmainutils ncdu unzip libudev-dev protobuf-compiler

2. 安装 Node.js + Yarn
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt install -y nodejs


启用 Corepack：

sudo corepack enable
sudo corepack prepare yarn@stable --activate


检查：

node -v
yarn -v

3. 安装 Rust
sudo curl https://sh.rustup.rs -sSf | sh
source $HOME/.cargo/env
rustup update


检查：

rustc --version

4. 安装 Solana CLI
curl --proto '=https' --tlsv1.2 -sSfL https://solana-install.solana.workers.dev | bash


加入 PATH：

echo 'export PATH="$HOME/.local/share/solana/install/active_release/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc


检查：

solana --version

5. 安装 Arcium 工具链
mkdir arcium-node
cd arcium-node

curl --proto '=https' --tlsv1.2 -sSfL https://arcium-install.arcium.workers.dev/ | bash


检查：

arcium --version
arcup --version

6. 生成 Keypair
节点 Keypair
solana-keygen new --outfile node-keypair.json --no-bip39-passphrase

Callback Keypair
solana-keygen new --outfile callback-kp.json --no-bip39-passphrase

身份 Keypair（PEM）
openssl genpkey -algorithm Ed25519 -out identity.pem

7. 切换 Solana 到 Devnet
solana config set --url https://api.devnet.solana.com

8. 领取 Devnet SOL

打开 Devnet Faucet：

🔗 https://faucet.solana.com/

为以下钱包分别领取 SOL：

node-keypair.json 的 pubkey

callback-kp.json 的 pubkey

检查余额：

solana balance <PUBKEY>

9. 初始化 ARX Node
arcium init-arx-accs \
  --keypair-path node-keypair.json \
  --callback-keypair-path callback-kp.json \
  --peer-keypair-path identity.pem \
  --node-offset <NODE_OFFSET> \
  --ip-address <YOUR_PUBLIC_IP> \
  --rpc-url https://api.devnet.solana.com


成功时你会看到：

Node activated
ARX node initialization complete!

10. 创建 node-config.toml
nano node-config.toml


填入：

[node]
offset = YOUR_NODE_OFFSET
hardware_claim = 0
starting_epoch = 0
ending_epoch = 9223372036854775807

[network]
address = "0.0.0.0"

[solana]
endpoint_rpc = "https://api.devnet.solana.com"
endpoint_wss = "wss://api.devnet.solana.com"
cluster = "Devnet"
commitment.commitment = "confirmed"


保存：Ctrl + X → Y → Enter

11. 创建 Docker Compose
nano docker-compose.yml


内容：

services:
  arx-node:
    image: arcium/arx-node
    container_name: arx-node
    environment:
      - NODE_IDENTITY_FILE=/usr/arx-node/node-keys/node_identity.pem
      - NODE_KEYPAIR_FILE=/usr/arx-node/node-keys/node_keypair.json
      - OPERATOR_KEYPAIR_FILE=/usr/arx-node/node-keys/operator_keypair.json
      - CALLBACK_AUTHORITY_KEYPAIR_FILE=/usr/arx-node/node-keys/callback_authority_keypair.json
      - NODE_CONFIG_PATH=/usr/arx-node/arx/node_config.toml
    volumes:
      - ./node-config.toml:/usr/arx-node/arx/node_config.toml
      - ./node-keypair.json:/usr/arx-node/node-keys/node_keypair.json:ro
      - ./node-keypair.json:/usr/arx-node/node-keys/operator_keypair.json:ro
      - ./callback-kp.json:/usr/arx-node/node-keys/callback_authority_keypair.json:ro
      - ./identity.pem:/usr/arx-node/node-keys/node_identity.pem:ro
      - ./arx-node-logs:/usr/arx-node/logs
    ports:
      - "8080:8080"
    restart: unless-stopped

12. 启动节点
docker compose up -d


查看容器：

docker ps

13. 查看运行日志（最重要）

列出日志文件：

docker exec -it arx-node ls /usr/arx-node/logs


查看日志：

docker exec -it arx-node tail -f /usr/arx-node/logs/<LOG_FILENAME>


你应看到：

Node activated
Coordinator received activation unit message: activated

14. 验证节点状态
arcium arx-active <NODE_OFFSET> --rpc-url https://api.devnet.solana.com


返回：

true


查看节点信息：

arcium arx-info <NODE_OFFSET> --rpc-url https://api.devnet.solana.com

FAQ: 关于 Cluster 功能

当前 Arcium Devnet 的 Cluster 功能：

可创建 cluster

但 devnet 上不一定显示任何 membership

这并不影响节点正常运行

官方目前未对 cluster 执行任务做开放

节点只要显示：

Node activated = true


就说明运行正常。

🎉 部署完成！

你的 Arcium 节点现在已正式上线并激活，支持自动重启、自动上链、可长期运行。
