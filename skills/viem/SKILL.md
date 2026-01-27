---
name: viem
description: Use viem for Celo development. Includes fee currency support, transaction signing, and Celo-specific configurations.
license: Apache-2.0
metadata:
  author: celo-org
  version: "1.0.0"
---

# Viem for Celo

Viem is a lightweight TypeScript library with first-class Celo support. It powers wagmi and RainbowKit.

Source: https://docs.celo.org/tooling/libraries-sdks/viem/index.md

## When to Use

- Building dApps that interact with Celo
- Sending transactions with alternative fee currencies
- Reading contract state and blockchain data
- Signing messages and transactions

## Installation

```bash
npm install viem
```

## Chain Configuration

### Mainnet

```typescript
import { celo } from "viem/chains";

// Chain ID: 42220
// RPC: https://forno.celo.org
// Explorer: https://celoscan.io
```

### Testnet (Alfajores)

```typescript
import { celoAlfajores } from "viem/chains";

// Chain ID: 44787
// RPC: https://alfajores-forno.celo-testnet.org
// Explorer: https://celo-alfajores.blockscout.com
```

Source: https://github.com/wevm/viem (chain definitions)

## Client Setup

### Public Client (Read Operations)

```typescript
import { createPublicClient, http } from "viem";
import { celo } from "viem/chains";

const publicClient = createPublicClient({
  chain: celo,
  transport: http(),
});

// Read contract state
const balance = await publicClient.getBalance({
  address: "0x...",
});
```

### Wallet Client (Write Operations)

```typescript
import { createWalletClient, custom } from "viem";
import { celo } from "viem/chains";

const walletClient = createWalletClient({
  chain: celo,
  transport: custom(window.ethereum),
});

const [address] = await walletClient.getAddresses();
```

## Fee Currency Support

Celo allows paying gas fees in tokens other than CELO. Pass `feeCurrency` to transaction functions.

Source: https://docs.celo.org/developer/fee-currency

### Fee Currency Addresses - Mainnet

| Token | Adapter Address |
|-------|-----------------|
| USDC | 0x2F25deB3848C207fc8E0c34035B3Ba7fC157602B |
| USDT | 0x0e2a3e05bc9a16f5292a6170456a710cb89c6f72 |

### Fee Currency Addresses - Celo Sepolia

| Token | Adapter Address |
|-------|-----------------|
| USDC | 0x4822e58de6f5e485eF90df51C41CE01721331dC0 |

### Transaction with Fee Currency

```typescript
import { createWalletClient, custom, parseEther } from "viem";
import { celo } from "viem/chains";

const USDC_ADAPTER = "0x2F25deB3848C207fc8E0c34035B3Ba7fC157602B";

const walletClient = createWalletClient({
  chain: celo,
  transport: custom(window.ethereum),
});

const [address] = await walletClient.getAddresses();

// Send transaction paying gas in USDC
const hash = await walletClient.sendTransaction({
  account: address,
  to: "0x...",
  value: parseEther("1"),
  feeCurrency: USDC_ADAPTER,
});
```

### Contract Call with Fee Currency

```typescript
const hash = await walletClient.writeContract({
  address: CONTRACT_ADDRESS,
  abi: CONTRACT_ABI,
  functionName: "transfer",
  args: [recipient, amount],
  feeCurrency: USDC_ADAPTER,
});
```

## Gas Price for Fee Currency

When using alternative fee currencies, fetch gas price denominated in that token:

```typescript
import { hexToBigInt } from "viem";

async function getGasPrice(
  client: PublicClient,
  feeCurrencyAddress?: `0x${string}`
): Promise<bigint> {
  const priceHex = await client.request({
    method: "eth_gasPrice",
    params: feeCurrencyAddress ? [feeCurrencyAddress] : [],
  });
  return hexToBigInt(priceHex);
}

// Get gas price in USDC
const gasPriceInUSDC = await getGasPrice(publicClient, USDC_ADAPTER);
```

## Reading Contract Data

```typescript
import { createPublicClient, http } from "viem";
import { celo } from "viem/chains";

const publicClient = createPublicClient({
  chain: celo,
  transport: http(),
});

const ERC20_ABI = [
  {
    name: "balanceOf",
    type: "function",
    stateMutability: "view",
    inputs: [{ name: "account", type: "address" }],
    outputs: [{ type: "uint256" }],
  },
] as const;

const balance = await publicClient.readContract({
  address: "0x765de816845861e75a25fca122bb6898b8b1282a", // USDm
  abi: ERC20_ABI,
  functionName: "balanceOf",
  args: ["0x..."],
});
```

## Multicall

Batch multiple read calls efficiently:

```typescript
const results = await publicClient.multicall({
  contracts: [
    {
      address: TOKEN_ADDRESS,
      abi: ERC20_ABI,
      functionName: "balanceOf",
      args: [userAddress],
    },
    {
      address: TOKEN_ADDRESS,
      abi: ERC20_ABI,
      functionName: "totalSupply",
    },
  ],
});
```

## Important Notes

- Viem is the recommended library for Celo (ethers.js and web3.js don't support `feeCurrency`)
- Fee currency transactions use type `0x7b` (CIP-64)
- Transactions with fee currencies incur ~50,000 additional gas
- Omitting `feeCurrency` defaults to paying in CELO

## Dependencies

```json
{
  "dependencies": {
    "viem": "^2.0.0"
  }
}
```

## Additional Resources

- [fee-currencies.md](references/fee-currencies.md) - Complete fee currency reference
