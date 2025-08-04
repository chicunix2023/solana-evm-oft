# Solana-EVM OFT 跨链代币项目操作指南

## 项目概述

本项目是一个基于 LayerZero 协议的跨链可替代代币 (OFT) 示例，主要实现 Solana 和 EVM 链之间的代币跨链转移功能。

## 前置知识要求

- [什么是 OFT(全链可替代代币)？](https://docs.layerzero.network/v2/concepts/applications/oft-standard)
- [什么是 OApp(全链应用)？](https://docs.layerzero.network/v2/concepts/applications/oapp-standard)
- 基本的 Solana 和以太坊开发知识

## 官方文档

- [Solana-EVM Omnichain Fungible Token (OFT) Example](https://github.com/LayerZero-Labs/devtools/tree/main/examples/oft-solana)

注意，官方文档中的示例代码可能与本项目有所不同。官方文档中的代码可能包含错误或不明确之处。

## 环境要求

### 必需组件

- **Rust**
- **Solana** `1.17.31`, `1.18.26`
- **Anchor** `0.29`
- **Node.js** `>=18.16.0`
- **pnpm** (推荐)

## 一、环境安装步骤

### 1. 安装 Rust

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y
```

### 2. 安装 Solana `1.17.31`

```bash
sh -c "$(curl -sSfL https://release.anza.xyz/v1.17.31/install)"
```

验证安装:

```bash
solana --version
```

### 3. 安装 Anchor `0.29`

```bash
cargo install --git https://github.com/coral-xyz/anchor --tag v0.29.0 anchor-cli --locked
```

## 二、项目初始化

### 1. 环境变量配置

复制环境变量模板:

```bash
cp .env.example .env
```

编辑 `.env` 文件，配置以下参数:

#### Solana 部署者配置

选择以下三种方式之一:

- **默认方式**: 使用 `~/.config/solana/id.json` (无需额外配置)
- **私钥方式**: 设置 `SOLANA_PRIVATE_KEY` (base58 字符串格式或 Uint8 数组格式)
- **密钥对路径**: 设置 `SOLANA_KEYPAIR_PATH`

#### Solana RPC 配置

设置 `RPC_URL_SOLANA` 或 `RPC_URL_SOLANA_TESTNET` 值。

```
RPC_URL_SOLANA=https://api.mainnet-beta.solana.com
RPC_URL_SOLANA_TESTNET="https://api.devnet.solana.com
```

#### EVM 部署者配置

```
MNEMONIC=test test test test test test test test test test test junk
# 或者
PRIVATE_KEY=0xabc...def
```

#### EVM RPC 配置

设置 `RPC_URL_ETHEREUM` 或 `RPC_URL_SEPOLIA` 值。默认值

### 2. 安装依赖

```bash
pnpm install
```

### 3. 账户资金准备

#### Solana 账户充值

为您的 Solana 部署者地址充值 SOL 代币。Devnet 上可以使用以下命令获取 SOL:

```bash
solana airdrop 5 -u devnet
```

#### EVM 账户充值

为您的 EVM 部署者地址充值相应测试网的原生代币。

### 4. 检查链的参数配置

确保 `layerzero.config.ts` 中的链参数配置正确。特别是 `chainId` 和 `endpointId` 的对应关系。

确保 `hardhat.config.ts` 中的链参数配置正确。

## 三、构建和部署

### 1. 准备 Solana OFT 程序密钥对

生成 OFT `<OFT_PROGRAM_ID>`:

```bash
anchor keys sync -p oft
```

查看生成的 `<OFT_PROGRAM_ID>`:

```bash
anchor keys list
```

会得到以下输出:

```text
endpoint: H3SKp4cL5rpzJDntDa2umKE9AHkGiyss1W8BNDndhHWp
oft: DLZdefiak8Ur82eWp3Fii59RiCRZn3SjNCmweCdhf1DD
```

记录输出中的 `oft` 作为后面要用到的 `<OFT_PROGRAM_ID>`。

确保 `<OFT_PROGRAM_ID>` 在项目中的配置正确。检查 `Anchor.toml` 中的 `oft` 和 `programs/oft/src/lib.rs` 中的 `declare_id!` 是否与生成的 `<OFT_PROGRAM_ID>` 匹配。

### 2. 构建 Solana OFT 程序

```bash
anchor build -p oft -e OFT_ID=<OFT_PROGRAM_ID>
```

```bash
solana rent $(wc -c < target/deploy/oft.so)
```

### 3. 部署 Solana OFT 程序

#### 切换到 Solana v1.18.26 (用于部署优化)

```bash
sh -c "$(curl -sSfL https://release.anza.xyz/v1.18.26/install)"
```

#### 执行部署 (推荐使用优先费用)

在 mainnet-beta 上部署:

```bash
solana program deploy --program-id target/deploy/oft-keypair.json target/deploy/oft.so -u mainnet-beta --with-compute-unit-price <COMPUTE_UNIT_PRICE_IN_MICRO_LAMPORTS>
```

在 devnet 上部署:

```bash
solana program deploy --program-id target/deploy/oft-keypair.json target/deploy/oft.so -u devnet --with-compute-unit-price <COMPUTE_UNIT_PRICE_IN_MICRO_LAMPORTS>
```

关于 `--with-compute-unit-price` 的说明:

- `--with-compute-unit-price`: 设置优先费用，单位为 micro-lamports (百万分之一的 lamport)

- 参考值见 [<COMPUTE_UNIT_PRICE_IN_MICRO_LAMPORTS>](https://www.quicknode.com/gas-tracker/solana)

- Devnet 上的参考值通常较低，但在主网可能需要更高的优先费用。

#### 切换回 Solana v1.17.31 (重要‼️)

```bash
sh -c "$(curl -sSfL https://release.anza.xyz/v1.17.31/install)"
```

### 4. 创建 Solana OFT

在 `Mainnet` 上

```bash
pnpm hardhat lz:oft:solana:create --eid 30168 --program-id <OFT_PROGRAM_ID> --name <OFT_NAME> --symbol <OFT_SYMBOL> --only-oft-store true
```

在 `Devnet` 上

```bash
pnpm hardhat lz:oft:solana:create --eid 40168 --program-id <OFT_PROGRAM_ID> --name <OFT_NAME> --symbol <OFT_SYMBOL> --only-oft-store true
```

替换命令中的 `OFT_PROGRAM_ID`、`OFT_NAME` 和 `OFT_SYMBOL` 为实际值。此命令将创建一个 Solana OFT，只有 OFT Store 作为 Mint Authority。

### 5. 部署 EVM 链 OFT 对等合约

将已经部署好的 `MyOFTAdapter` 项目中的 `deployments` 文件夹中的 Evm 合约部分复制到本项目中。

## 四、启用跨链消息传递

### 1. 初始化配置账户

```bash
npx hardhat lz:oft:solana:init-config --oapp-config layerzero.config.ts
```

### 2. 配置跨链连接(Wiring)

```bash
pnpm hardhat lz:oapp:wire --oapp-config layerzero.config.ts
```

## 五、跨链代币转移

### 从 Ethereuem 发送到 Solana

```bash
npx hardhat lz:oft:send --src-eid 30101 --dst-eid 30168 --to <SOLANA_ADDRESS> --amount 1
```

### 从 Solana 发送到 Ethereuem

```bash
npx hardhat lz:oft:send --src-eid 30168 --dst-eid 30101 --to <EVM_ADDRESS> --amount 1
```

### 从 Ethereuem Sepolia 发送到 Solana Devnet

```bash
npx hardhat lz:oft:send --src-eid 40161 --dst-eid 40168 --to <SOLANA_ADDRESS> --amount 1
```

### 从 Solana Devnet 发送到 Ethereuem Sepolia

```bash
npx hardhat lz:oft:send --src-eid 40168 --dst-eid 40161 --to <EVM_ADDRESS> --amount 1
```

**端点 ID 说明**:

- `30101`: Ethereum Mainnet
- `30168`: Solana Mainnet
- `40161`: Ethereum Sepolia
- `40168`: Solana Devnet

成功发送后，脚本会提供 LayerZero Scan 上的消息链接，您可以通过点击目标交易哈希来验证 OFT 是否成功发送。

## 下一步骤

成功部署 OFT 后，可以考虑:

1. 学习 [安全堆栈](https://docs.layerzero.network/v2/developers/evm/protocol-gas-settings/security-stack)
2. 了解 [消息执行选项](https://docs.layerzero.network/v2/developers/evm/protocol-gas-settings/options)
3. 添加其他链的支持
4. 配置生产环境的安全设置
