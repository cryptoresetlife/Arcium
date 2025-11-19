# Arcium 节点部署教程

## 1. 安装系统依赖
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl iptables build-essential git wget lz4 jq make gcc nano automake autoconf tmux htop nvme-cli pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip libudev-dev protobuf-compiler
```

## 2. 安装 Node.js 与 Yarn
```bash
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt install -y nodejs
sudo corepack enable
sudo corepack prepare yarn@stable --activate
node -v
yarn -v
```

## 3. 安装 Rust
```bash
sudo curl https://sh.rustup.rs -sSf | sh
source $HOME/.cargo/env
rustup update
rustc --version
```

## 4. 安装 Solana CLI
```bash
curl --proto '=https' --tlsv1.2 -sSfL https://solana-install.solana.workers.dev | bash
echo 'export PATH="$HOME/.local/share/solana/install/active_release/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
solana --version
```

## 5. 安装 Arcium 工具链
```bash
mkdir arcium-node
cd arcium-node
curl --proto '=https' --tlsv1.2 -sSfL https://arcium-install.arcium.workers.dev/ | bash
arcium --version
arcup --version
```

## 6. 生成节点密钥
```bash
solana-keygen new --outfile node-keypair.json --no-bip39-passphrase
solana-keygen new --outfile callback-kp.json --no-bip39-passphrase
openssl genpkey -algorithm Ed25519 -out identity.pem
```

## 7. 切换 Devnet
```bash
solana config set --url https://api.devnet.solana.com
```

## 8. 初始化 ARX Node
```bash
arcium init-arx-accs \
  --keypair-path node-keypair.json \
  --callback-keypair-path callback-kp.json \
  --peer-keypair-path identity.pem \
  --node-offset <NODE_OFFSET> \
  --ip-address <PUBLIC_IP> \
  --rpc-url https://api.devnet.solana.com
```

## 9. 创建 node-config.toml
```toml
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
```

## 10. 创建 docker-compose.yml
```yaml
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
```

## 11. 启动节点
```bash
docker compose up -d
docker ps
```

## 12. 查看日志
```bash
docker exec -it arx-node ls /usr/arx-node/logs
docker exec -it arx-node tail -f /usr/arx-node/logs/<LOG_FILENAME>
```
<img width="1108" height="367" alt="image" src="https://github.com/user-attachments/assets/f25342c6-f448-4bae-945e-b7b261d60b6f" />

## 13. 查询节点状态
```bash
arcium arx-active <NODE_OFFSET> --rpc-url https://api.devnet.solana.com
arcium arx-info <NODE_OFFSET> --rpc-url https://api.devnet.solana.com
```
<img width="899" height="206" alt="image" src="https://github.com/user-attachments/assets/3672f0ab-57c7-4143-aa22-99bf669f07b1" />

