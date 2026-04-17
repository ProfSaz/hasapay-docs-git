# Welcome to HasaPay

**Enterprise Stablecoin Wallet Infrastructure for Fintechs**

HasaPay provides a complete Wallet-as-a-Service (WaaS) platform that enables businesses to integrate multi-chain cryptocurrency wallets into their applications with a simple API.

---

## Why HasaPay?

| Feature | Description |
|---------|-------------|
| 🔐 **Non-Custodial Security** | Your keys, your crypto. We never store private keys on our servers. |
| ⚡ **Multi-Chain Support** | Ethereum, Polygon, Tron, Base, BSC, Solana, Bitcoin, and Aptos. |
| 🚀 **Simple Integration** | RESTful API with comprehensive SDKs. Go live in hours, not weeks. |

---

## Quick Links

| Resource | Description |
|----------|-------------|
| [Quick Start](documentation/quickstart.md) | Get your first wallet created in 5 minutes |
| [Authentication](documentation/authentication.md) | Learn about JWT and HMAC authentication |
| [API Reference](api-reference/overview.md) | Browse all available endpoints |

---

## Get Started in 5 Minutes

### Step 1: Create an Account

Sign up at [dashboardtest.hasapay.com](https://dashboardtest.hasapay.com) to get your API credentials.

### Step 2: Get Your API Keys

After email verification, you'll receive:
- **API Key** - For identifying your organization
- **Secret Key** - For signing HMAC requests (keep this safe!)

### Step 3: Make Your First API Call

```bash
curl -X POST https://apitest.hasapay.com/api/v1/wallets \
  -H "Content-Type: application/json" \
  -H "X-API-Key: your_api_key" \
  -H "X-Timestamp: $(date +%s)" \
  -H "X-Request-ID: $(uuidgen)" \
  -H "X-Signature: your_hmac_signature" \
  -d '{
    "chain": "ethereum",
    "network": "sepolia",
    "label": "My First Wallet"
  }'
```

### Step 4: Start Building

Use our comprehensive API to:
- ✅ Create HD wallets across multiple chains
- ✅ Generate unlimited deposit addresses
- ✅ Monitor incoming deposits in real-time
- ✅ Send transactions programmatically
- ✅ Track balances and transaction history

---

## Supported Chains

| Chain | Mainnet | Testnet | Native Token |
|-------|---------|---------|--------------|
| Ethereum | ✅ | ✅ Sepolia | ETH |
| Polygon | ✅ | ✅ Amoy | MATIC |
| Tron | ✅ | ✅ Shasta | TRX |
| Base | ✅ | ✅ Sepolia | ETH |
| BSC | ✅ | ✅ Testnet | BNB |
| Solana | ✅ | ✅ Devnet | SOL |
| Bitcoin | ✅ | ✅ Testnet | BTC |
| Aptos | ✅ | ✅ Testnet | APT |

---

## Supported Assets

HasaPay supports native tokens and popular stablecoins on each chain:

- **USDT** - Tether USD
- **USDC** - USD Coin  
- **DAI** - Dai Stablecoin
- **BUSD** - Binance USD
- **And more...** - Enable any ERC-20/TRC-20/SPL token

---

## Authentication Methods

| Method | Used For | Headers |
|--------|----------|---------|
| **JWT** | Dashboard, team, settings | `Authorization: Bearer <token>` |
| **HMAC** | Wallets, transactions, addresses | `X-API-Key`, `X-Timestamp`, `X-Signature`, `X-Request-ID` |

---

## Testnet vs Mainnet

| Feature | Testnet (Sandbox) | Mainnet |
|---------|-------------------|---------|
| Real Funds | ❌ Test tokens only | ✅ Real crypto |
| KYC Required | ❌ No | ✅ Yes |
| Rate Limits | Lower | Higher |

> 💡 **Tip:** Start with Testnet to build and test your integration, then upgrade to Mainnet when ready for production.

---

## Need Help?

- 📧 **Email:** support@hasapay.com
- 📚 **Documentation:** You're here!

---

## Ready to Build?

[**Sign Up Free →**](https://dashboardtest.hasapay.com/register) | [**View API Reference →**](api-reference/overview.md)
