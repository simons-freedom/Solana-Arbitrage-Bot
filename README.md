### Overview
Solana Arbitrage Bot is a TypeScript-based arbitrage bot that utilizes the Jupiter API to execute arbitrage trades on the Solana blockchain network. The bot continuously monitors market price differences and automatically executes buy-low-sell-high operations.

### Features
- **Real-time Quote Retrieval**: Obtains the latest quotes for WSOL and USDC through the Jupiter API
- **Automated Arbitrage Trading**: Automatically executes trades when profitable price differences are detected
- **Jito Tip Mechanism**: Automatically calculates and pays Jito tips to increase transaction priority
- **Transaction Bundle Submission**: Sends transaction bundles through Jito's block engine
- **Continuous Monitoring**: Checks for arbitrage opportunities every 200 milliseconds

### Tech Stack
- **Core Library**: @solana/web3.js
- **HTTP Client**: axios
- **Environment Management**: dotenv
- **Development Tool**: TypeScript

### Arbitrage Principle
1. **Bi-directional Quote Query**:
   - First query the exchange rate from WSOL → USDC
   - Then query the rate from USDC → WSOL using the obtained USDC amount
2. **Profit Calculation**: Compare the final WSOL amount with the initial investment
3. **Transaction Execution**: Execute arbitrage when profit exceeds the threshold (3000 lamports)

### Core Code Analysis

#### Initialization Configuration
```typescript
// Wallet and connection initialization
const payer = getKeypairFromEnvironment("SECRET_KEY");
const connection = new Connection('https://solana-rpc.publicnode.com', 'processed');

// Token addresses
const wSolMint = 'So11111111111111111111111111111111111111112';
const usdcMint = 'EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v';
```

#### Arbitrage Logic

```typescript
// Get bi-directional quotes
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

// Calculate profit
const diffLamports = quote1Resp.data.outAmount - 10000000;
if (diffLamports > 3000) {
    // Execute arbitrage...
}
```


#### Jito Tip Payment

```typescript
const jitoTip = Math.floor(diffLamports * 0.5);
const tipInstruction = SystemProgram.transfer({
    fromPubkey: payer.publicKey,
    toPubkey: new PublicKey('Jito tip address'),
    lamports: jitoTip
});
```

## Requirements
- Node.js environment
- Solana wallet private key (configured in .env file)
- Sufficient SOL balance to pay transaction fees


## Installation and Running
### Install Dependencies:

```bash
npm install
```

### Configure Environment Variables
Create an .env file and add:

```plaintext
SECRET_KEY=your_private_key
```

### Start the Bot:

```bash
ts-node src/main.ts
```
