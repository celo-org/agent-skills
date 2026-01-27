---
name: contract-verification
description: Verify smart contracts on Celo. Use when publishing contract source code to Celoscan or Blockscout.
license: Apache-2.0
metadata:
  author: celo-org
  version: "1.0.0"
---

# Contract Verification on Celo

This skill covers verifying smart contracts on Celo block explorers, making source code publicly readable.

## When to Use

- After deploying a contract to Celo
- Publishing open-source contracts
- Enabling contract interaction via explorer UI
- Building trust with users

## Verification Methods

| Method | Best For |
|--------|----------|
| Hardhat | Automated deployment workflows |
| Foundry | Foundry-based projects |
| Celoscan UI | Quick manual verification |
| Blockscout | Alternative explorer |

## Hardhat Verification

### Configuration

```javascript
// hardhat.config.js
require("dotenv").config();
require("@nomicfoundation/hardhat-verify");

module.exports = {
  solidity: "0.8.28",
  networks: {
    celo: {
      url: "https://forno.celo.org",
      accounts: [process.env.PRIVATE_KEY],
      chainId: 42220,
    },
    celoSepolia: {
      url: "https://forno.celo-sepolia.celo-testnet.org",
      accounts: [process.env.PRIVATE_KEY],
      chainId: 11142220,
    },
  },
  etherscan: {
    apiKey: {
      celo: process.env.ETHERSCAN_API_KEY,
      celoSepolia: process.env.ETHERSCAN_API_KEY,
    },
    customChains: [
      {
        network: "celo",
        chainId: 42220,
        urls: {
          apiURL: "https://api.celoscan.io/api",
          browserURL: "https://celoscan.io",
        },
      },
      {
        network: "celoSepolia",
        chainId: 11142220,
        urls: {
          apiURL: "https://api-sepolia.celoscan.io/api",
          browserURL: "https://sepolia.celoscan.io",
        },
      },
    ],
  },
};
```

### Environment Variables

```bash
# .env
PRIVATE_KEY=your_private_key
ETHERSCAN_API_KEY=your_etherscan_api_key
```

Get an API key from [Etherscan](https://etherscan.io/myapikey).

### Verify Commands

**Mainnet:**
```bash
npx hardhat verify --network celo <CONTRACT_ADDRESS> <CONSTRUCTOR_ARGS>
```

**Testnet:**
```bash
npx hardhat verify --network celoSepolia <CONTRACT_ADDRESS> <CONSTRUCTOR_ARGS>
```

### Example with Constructor Arguments

```bash
# Contract with constructor: constructor(string memory name, uint256 value)
npx hardhat verify --network celo 0x1234...5678 "MyToken" 1000000
```

### Complex Constructor Arguments

For complex arguments, create a file:

```javascript
// arguments.js
module.exports = [
  "MyToken",
  "MTK",
  1000000,
  "0x1234567890123456789012345678901234567890",
];
```

```bash
npx hardhat verify --network celo 0x1234...5678 --constructor-args arguments.js
```

## Foundry Verification

### Configuration

```toml
# foundry.toml
[profile.default]
src = "src"
out = "out"
libs = ["lib"]
solc_version = "0.8.28"

[rpc_endpoints]
celo = "https://forno.celo.org"
celoSepolia = "https://forno.celo-sepolia.celo-testnet.org"
```

### Verify Commands

**Mainnet:**
```bash
forge verify-contract \
  --chain-id 42220 \
  --etherscan-api-key $ETHERSCAN_API_KEY \
  <CONTRACT_ADDRESS> \
  src/MyContract.sol:MyContract \
  --watch
```

**Testnet:**
```bash
forge verify-contract \
  --chain-id 11142220 \
  --etherscan-api-key $ETHERSCAN_API_KEY \
  <CONTRACT_ADDRESS> \
  src/MyContract.sol:MyContract \
  --watch
```

### With Constructor Arguments

```bash
forge verify-contract \
  --chain-id 42220 \
  --etherscan-api-key $ETHERSCAN_API_KEY \
  <CONTRACT_ADDRESS> \
  src/MyContract.sol:MyContract \
  --constructor-args $(cast abi-encode "constructor(string,uint256)" "MyToken" 1000000) \
  --watch
```

### Deploy and Verify in One Command

```bash
forge create \
  --rpc-url https://forno.celo.org \
  --private-key $PRIVATE_KEY \
  --etherscan-api-key $ETHERSCAN_API_KEY \
  --verify \
  src/MyContract.sol:MyContract \
  --constructor-args "MyToken" 1000000
```

## Celoscan UI Verification

1. Go to your contract on [Celoscan](https://celoscan.io)
2. Click the **Contract** tab
3. Click **Verify & Publish**
4. Select compiler settings:
   - Compiler Type (Solidity Single/Multi-file)
   - Compiler Version
   - License Type
5. Paste your source code
6. Add constructor arguments (ABI-encoded)
7. Complete CAPTCHA
8. Click **Verify and Publish**

## Blockscout Verification

Celo Explorer (Blockscout) also supports verification:

1. Go to [Celo Explorer](https://explorer.celo.org)
2. Find your contract
3. Click **Code** tab
4. Click **Verify & Publish**
5. Follow similar steps as Celoscan

## Troubleshooting

### "Unable to verify"

**Causes:**
- Compiler version mismatch
- Optimizer settings mismatch
- Wrong constructor arguments

**Solutions:**
- Match exact compiler version used in deployment
- Match optimizer settings (enabled, runs)
- Verify constructor args are ABI-encoded correctly

### "Already verified"

Contract is already verified. Check the explorer to see the source code.

### "Contract not found"

**Causes:**
- Wrong contract address
- Contract not yet indexed

**Solutions:**
- Double-check the address
- Wait a few minutes for indexing

### Proxy Contracts

For proxy contracts, verify both:
1. The implementation contract
2. The proxy contract

Then link them on Celoscan:
1. Go to proxy contract
2. Click "More Options"
3. Select "Is this a proxy?"
4. Follow verification steps

## API Endpoints

| Network | API URL |
|---------|---------|
| Mainnet | https://api.celoscan.io/api |
| Sepolia | https://api-sepolia.celoscan.io/api |

## Dependencies

For Hardhat:
```json
{
  "devDependencies": {
    "@nomicfoundation/hardhat-verify": "^2.0.0",
    "hardhat": "^2.19.0"
  }
}
```

## Additional Resources

- [verification-config.md](references/verification-config.md) - Complete configuration examples
