---
name: bridging
description: Bridge assets to and from Celo. Use when transferring tokens between Celo and other chains like Ethereum.
license: Apache-2.0
metadata:
  author: celo-org
  version: "1.0.0"
---

# Bridging to Celo

This skill covers bridging assets between Celo and other blockchains, including native bridges and third-party solutions.

## When to Use

- Transferring assets from Ethereum to Celo
- Bridging tokens from Celo to other chains
- Integrating cross-chain functionality
- Building multi-chain applications

## Bridge Options

### Native Bridges (L1 ↔ L2)

| Bridge | URL | Networks |
|--------|-----|----------|
| Superbridge | https://superbridge.app/celo | Mainnet, Sepolia |

Native bridges provide direct transfers between Celo L2 and Ethereum L1 without intermediaries.

### Third-Party Bridges

| Bridge | URL | Description |
|--------|-----|-------------|
| Squid Router | https://v2.app.squidrouter.com | Cross-chain routing via Axelar |
| Jumper Exchange | https://jumper.exchange | Multi-chain DEX |
| Portal (Wormhole) | https://portalbridge.com | Token bridge protocol |
| AllBridge | https://app.allbridge.io/bridge | EVM and non-EVM chains |
| Satellite (Axelar) | https://satellite.money | Axelar network bridge |
| Transporter (CCIP) | https://www.transporter.io | Chainlink CCIP |
| Layerswap | https://layerswap.io/app | 60+ chains and CEXs |
| SmolRefuel | https://smolrefuel.com | Gasless refueling |

## Native ETH Bridging

Bridge native ETH from Ethereum to Celo as WETH.

### Contract Addresses

**Ethereum Mainnet → Celo:**

| Contract | Address |
|----------|---------|
| SuperBridgeETHWrapper (L1) | 0x3bC7C4f8Afe7C8d514c9d4a3A42fb8176BE33c1e |
| L1 Standard Bridge | 0x9C4955b92F34148dbcfDCD82e9c9eCe5CF2badfe |
| L1 WETH | 0xc02aaa39b223fe8d0a0e5c4f27ead9083c756cc2 |
| L2 WETH (Celo) | 0xD221812de1BD094f35587EE8E174B07B6167D9Af |

**Sepolia Testnet → Celo Sepolia:**

| Contract | Address |
|----------|---------|
| SuperBridgeETHWrapper (L1) | 0x523e358dFd0c4e98F3401DAc7b1879445d377e37 |
| L1 Standard Bridge | 0xec18a3c30131a0db4246e785355fbc16e2eaf408 |
| L1 WETH | 0x7b79995e5f793A07Bc00c21412e50Ecae098E7f9 |
| L2 WETH (Celo Sepolia) | 0x2cE73DC897A3E10b3FF3F86470847c36ddB735cf |

### Bridge ETH to Celo

```typescript
import { createWalletClient, custom, parseEther } from "viem";
import { mainnet } from "viem/chains";

const SUPERBRIDGE_WRAPPER = "0x3bC7C4f8Afe7C8d514c9d4a3A42fb8176BE33c1e";

const WRAPPER_ABI = [
  {
    name: "wrapAndBridge",
    type: "function",
    stateMutability: "payable",
    inputs: [
      { name: "_minGasLimit", type: "uint32" },
      { name: "_extraData", type: "bytes" },
    ],
    outputs: [],
  },
] as const;

async function bridgeETHToCelo(amount: string): Promise<string> {
  const walletClient = createWalletClient({
    chain: mainnet,
    transport: custom(window.ethereum),
  });

  const [address] = await walletClient.getAddresses();

  const hash = await walletClient.writeContract({
    address: SUPERBRIDGE_WRAPPER,
    abi: WRAPPER_ABI,
    functionName: "wrapAndBridge",
    args: [200000, "0x"],
    value: parseEther(amount),
  });

  return hash;
}
```

## Natively Bridged Tokens

These tokens have official bridges from Ethereum:

| Token | L1 Address (Ethereum) | L2 Address (Celo) |
|-------|----------------------|-------------------|
| WETH | 0xc02aaa39b223fe8d0a0e5c4f27ead9083c756cc2 | 0xD221812de1BD094f35587EE8E174B07B6167D9Af |
| USDC | 0xa0b86991c6218b36c1d19d4a2e9eb0ce3606eb48 | 0xcebA9300f2b948710d2653dD7B07f33A8B32118C |
| USDT | 0xdac17f958d2ee523a2206206994597c13d831ec7 | 0x48065fbbe25f71c9282ddf5e1cd6d6a887483d5e |
| DAI | 0x6b175474e89094c44da98b954eedeac495271d0f | 0x90Ca507a5D4458a4C6C6249d186b6dCb02a5BCCd |
| WBTC | 0x2260fac5e5542a773aa44fbcfedf7c193bc2c599 | 0xBAAB46E28388d2779e6E31Fd00cF0e5Ad95E327B |
| AAVE | 0x7fc66500c84a76ad7e9c93437bfc5ac33e2ddae9 | 0x7fB0f7dE684FCd0F2D25Ec3bbf34e9BfA5B8B4E0 |
| LINK | 0x514910771af9ca656af840dff83e8264ecf986ca | 0xB1D77cD06acB1E7Ca01ca1fB1A4faFDC03A628ba |
| UNI | 0x1f9840a85d5af5bf1d1762f925bdaddc4201f984 | 0x71e26d0E519D14591b9dE9a0fE9513A398101490 |

## Cross-Chain Messaging Protocols

For building cross-chain dApps:

| Protocol | Mainnet | Testnet |
|----------|---------|---------|
| Chainlink CCIP | ✓ | - |
| Hyperlane | ✓ | ✓ (Sepolia) |
| Wormhole | ✓ | - |
| LayerZero | ✓ | - |
| Axelar | ✓ | - |

## Using LI.FI SDK

For cross-chain swaps and bridges:

```typescript
import { createConfig, getQuote, executeRoute } from "@lifi/sdk";

// Initialize LI.FI
createConfig({
  integrator: "your-app-name",
});

// Get bridge quote
const quote = await getQuote({
  fromChain: 1, // Ethereum
  toChain: 42220, // Celo
  fromToken: "0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48", // USDC on Ethereum
  toToken: "0xcebA9300f2b948710d2653dD7B07f33A8B32118C", // USDC on Celo
  fromAmount: "1000000000", // 1000 USDC (6 decimals)
  fromAddress: userAddress,
});

// Execute the bridge
const result = await executeRoute(quote, {
  updateRouteHook: (route) => {
    console.log("Route updated:", route);
  },
});
```

## Bridge Considerations

### Security
- Native bridges (Superbridge) are the most secure
- Third-party bridges rely on their own security models
- Always verify contract addresses before bridging

### Timing
- Native L1→L2 bridges: ~15-20 minutes
- L2→L1 withdrawals: 7 days (challenge period)
- Third-party bridges: varies (minutes to hours)

### Fees
- Native bridges: gas fees only
- Third-party bridges: gas + bridge fees

## Dependencies

```json
{
  "dependencies": {
    "viem": "^2.0.0"
  }
}
```

For LI.FI integration:

```json
{
  "dependencies": {
    "@lifi/sdk": "^3.0.0"
  }
}
```

## Additional Resources

- [bridge-contracts.md](references/bridge-contracts.md) - All bridge contract addresses
- [bridged-tokens.md](references/bridged-tokens.md) - Complete list of bridged tokens
