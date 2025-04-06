# Solana 套利机器人

### 概述
Solana Arbitrage Bot 是一个基于 TypeScript 开发的套利机器人，利用 Jupiter API 在 Solana 区块链网络上进行套利交易。该机器人通过持续监控市场价格差异，自动执行低买高卖操作。

### 功能特性
- **实时报价获取**：通过 Jupiter API 获取 WSOL 和 USDC 的最新报价
- **自动套利交易**：检测到有利可图的价差时自动执行交易
- **Jito 小费机制**：自动计算并支付 Jito 小费以提高交易优先级
- **交易捆绑发送**：通过 Jito 的区块引擎发送交易捆绑
- **循环监控**：每200毫秒检查一次套利机会

### 技术栈
- **核心库**：@solana/web3.js
- **HTTP客户端**：axios
- **环境管理**：dotenv
- **开发工具**：TypeScript

### 套利原理
1. **双向报价查询**：
   - 首先查询 WSOL → USDC 的兑换报价
   - 然后用获得的USDC数量查询 USDC → WSOL 的报价
2. **利润计算**：比较最终获得的WSOL数量与初始投入
3. **交易执行**：当利润超过阈值(3000 lamports)时执行套利

### 核心代码解析

#### 初始化配置
```typescript
// 钱包和连接初始化
const payer = getKeypairFromEnvironment("SECRET_KEY");
const connection = new Connection('https://solana-rpc.publicnode.com', 'processed');

// 代币地址
const wSolMint = 'So11111111111111111111111111111111111111112';
const usdcMint = 'EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v';
```

#### 套利逻辑

```typescript
// 获取双向报价
const quote0Resp = await axios.get(quoteUrl, { 
    params: {
        inputMint: wSolMint,
        outputMint: usdcMint,
        amount: 10000000 // 0.01 WSOL
    }
});

const quote1Resp = await axios.get(quoteUrl, {
    params: {
        inputMint: usdcMint,
        outputMint: wSolMint,
        amount: quote0Resp.data.outAmount
    }
});

// 计算利润
const diffLamports = quote1Resp.data.outAmount - 10000000;
if (diffLamports > 3000) {
    // 执行套利...
}
```


#### Jito 小费支付

```typescript
const jitoTip = Math.floor(diffLamports * 0.5);
const tipInstruction = SystemProgram.transfer({
    fromPubkey: payer.publicKey,
    toPubkey: new PublicKey('Jito小费地址'),
    lamports: jitoTip
});
```

## 运行要求
- Node js 环境
- Solana 钱包私钥（配置在.env文件）
- 足够的SOL余额支付交易费用


## 安装与运行
### 安装依赖：

```bash
npm install
```

### 配置环境变量
创建.env文件并添加：

```plainText
SECRET_KEY=您的私钥
```

### 启动机器人：

```bash
ts-node src/main.ts
```