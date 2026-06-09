# Welcome to HasaPay

**Enterprise Stablecoin Wallet Infrastructure for Fintechs**

HasaPay provides a complete Wallet-as-a-Service (WaaS) platform that enables businesses to integrate multi-chain cryptocurrency wallets into their applications with a simple API.

---

## Why HasaPay?

<table>
<tr>
<td>

### 🔐 Non-Custodial Security
Your keys, your crypto. We never store private keys on our servers.

</td>
<td>

### ⚡ Multi-Chain Support
Ethereum, Polygon, Tron, Base, BSC, Solana, Bitcoin, and Aptos — with per-chain config overrides for thresholds, fees, and sweep behavior.

</td>
<td>

### 🚀 Simple Integration
RESTful API with comprehensive SDKs. Go live in hours, not weeks.

</td>
</tr>
</table>

### Built-in revenue infrastructure

- **Auto-sweeping** — child-address funds consolidate to your master wallet on threshold, with per-chain overrides and manual triggers. Tron TRC-20 uses a fund-and-sweep flow that handles energy payment for you.
- **Fee engine** — five-tier resolution (address → org × chain × network × token → chain → platform), server-side estimates for deposits and withdrawals, per-period analytics and per-source ranking.
- **Webhooks** — signed, retried, with a queryable delivery log and manual retry.

---

## Quick Links

| Resource | Description |
|----------|-------------|
| [Quick Start](documentation/quickstart.md) | Get your first wallet created in 5 minutes |
| [Authentication](documentation/authentication.md) | JWT, HMAC, and dual-auth — pick the right tier per endpoint |
| [API Reference](api-reference/overview.md) | Browse all available endpoints |
| [Fees](api-reference/fees.md) | Configure fees, preview them server-side, and pull analytics |
| [Sweep](api-reference/sweep.md) | Auto-sweep config, per-chain overrides, manual triggers |
| [Postman Collection](https://www.postman.com/hasapay) | Import and test APIs instantly |

---

## Get Started in 5 Minutes

### Step 1: Create an Account

Sign up at [dashboardtest.hasapay.com](https://dashboardtest.hasapay.com) to get your API credentials.

### Step 2: Get Your API Keys

After email verification, you'll receive:
- **API Key** - For identifying your organization
- **Secret Key** - For signing HMAC requests (keep this safe!)

### Step 3: Make Your First API Call

HMAC writes require four headers — see [Authentication](documentation/authentication.md) for the full signing recipe.

```bash
# Create a wallet
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

## Testnet vs Mainnet

| Feature | Testnet (Sandbox) | Mainnet |
|---------|-------------------|---------|
| API Calls | 500/day | Unlimited |
| Wallets | 3 per chain | Unlimited |
| Child Addresses | 5 per wallet | Unlimited |
| Real Funds | ❌ Test tokens only | ✅ Real crypto |
| KYC Required | ❌ No | ✅ Yes |

> 💡 **Tip:** Start with Testnet to build and test your integration, then upgrade to Mainnet when ready for production.

---

## Need Help?

- 📧 **Email:** support@hasapay.com
- 📚 **Documentation:** You're here!
- 💬 **Discord:** [Join our community](https://discord.gg/hasapay)

---

## Ready to Build?

<table>
<tr>
<td align="center">

[**Sign Up Free →**](https://dashboardtest.hasapay.com/register)

Create your account and get API keys instantly

</td>
<td align="center">

[**View API Reference →**](api-reference/overview.md)

Explore all available endpoints

</td>
</tr>
</table>
