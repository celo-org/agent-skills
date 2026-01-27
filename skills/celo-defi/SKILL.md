---
name: celo-defi
description: Integrate DeFi protocols on Celo. Use when building swaps, lending, or liquidity applications with Uniswap, Aave, or other DeFi protocols.
license: Apache-2.0
metadata:
  author: celo-org
  version: "1.0.0"
---

# Celo DeFi Integration

This skill covers integrating DeFi protocols on Celo, including Uniswap for swaps and Aave for lending/borrowing.

## When to Use

- Building swap functionality with Uniswap
- Integrating lending/borrowing with Aave
- Creating liquidity provision features
- Building DeFi aggregators

## DeFi Protocols on Celo

### Decentralized Exchanges
- **Uniswap V3/V4** - Primary DEX with concentrated liquidity
- **Velodrome** - Low-fee swaps with liquidity mining
- **Ubeswap** - Mobile-optimized exchange

### Lending & Borrowing
- **Aave V3** - Decentralized lending (launched March 2025)

### DEX Aggregators
- **LI.FI** - Cross-chain routing
- **Jumper Exchange** - Multi-chain swaps

## Uniswap V3 Integration

### Contract Addresses (Mainnet)

| Contract | Address |
|----------|---------|
| Factory | 0xAfE208a311B21f13EF87E33A90049fC17A7acDEc |
| SwapRouter02 | 0x5615CDAb10dc425a742d643d949a7F474C01abc4 |
| QuoterV2 | 0x82825d0554fA07f7FC52Ab63c961F330fdEFa8E8 |
| NonfungiblePositionManager | 0x3d79EdAaBC0EaB6F08ED885C05Fc0B014290D95A |
| UniversalRouter | 0x643770E279d5D0733F21d6DC03A8efbABf3255B4 |
| Permit2 | 0x000000000022D473030F116dDEE9F6B43aC78BA3 |

### Simple Swap Example

```typescript
import {
  createPublicClient,
  createWalletClient,
  custom,
  http,
  parseUnits,
  encodeFunctionData,
} from "viem";
import { celo } from "viem/chains";

const SWAP_ROUTER = "0x5615CDAb10dc425a742d643d949a7F474C01abc4";
const USDM_ADDRESS = "0x765de816845861e75a25fca122bb6898b8b1282a";
const CELO_ADDRESS = "0x471EcE3750Da237f93B8E339c536989b8978a438";

const SWAP_ROUTER_ABI = [
  {
    name: "exactInputSingle",
    type: "function",
    stateMutability: "payable",
    inputs: [
      {
        name: "params",
        type: "tuple",
        components: [
          { name: "tokenIn", type: "address" },
          { name: "tokenOut", type: "address" },
          { name: "fee", type: "uint24" },
          { name: "recipient", type: "address" },
          { name: "amountIn", type: "uint256" },
          { name: "amountOutMinimum", type: "uint256" },
          { name: "sqrtPriceLimitX96", type: "uint160" },
        ],
      },
    ],
    outputs: [{ type: "uint256" }],
  },
] as const;

async function swapExactInput(
  tokenIn: string,
  tokenOut: string,
  amountIn: bigint,
  amountOutMin: bigint,
  fee: number = 3000 // 0.3%
): Promise<string> {
  const walletClient = createWalletClient({
    chain: celo,
    transport: custom(window.ethereum),
  });

  const [address] = await walletClient.getAddresses();

  const hash = await walletClient.sendTransaction({
    account: address,
    to: SWAP_ROUTER,
    data: encodeFunctionData({
      abi: SWAP_ROUTER_ABI,
      functionName: "exactInputSingle",
      args: [
        {
          tokenIn: tokenIn as `0x${string}`,
          tokenOut: tokenOut as `0x${string}`,
          fee,
          recipient: address,
          amountIn,
          amountOutMinimum: amountOutMin,
          sqrtPriceLimitX96: 0n,
        },
      ],
    }),
  });

  return hash;
}
```

### Get Quote

```typescript
const QUOTER_V2 = "0x82825d0554fA07f7FC52Ab63c961F330fdEFa8E8";

const QUOTER_ABI = [
  {
    name: "quoteExactInputSingle",
    type: "function",
    stateMutability: "nonpayable",
    inputs: [
      {
        name: "params",
        type: "tuple",
        components: [
          { name: "tokenIn", type: "address" },
          { name: "tokenOut", type: "address" },
          { name: "amountIn", type: "uint256" },
          { name: "fee", type: "uint24" },
          { name: "sqrtPriceLimitX96", type: "uint160" },
        ],
      },
    ],
    outputs: [
      { name: "amountOut", type: "uint256" },
      { name: "sqrtPriceX96After", type: "uint160" },
      { name: "initializedTicksCrossed", type: "uint32" },
      { name: "gasEstimate", type: "uint256" },
    ],
  },
] as const;

async function getQuote(
  tokenIn: string,
  tokenOut: string,
  amountIn: bigint,
  fee: number = 3000
): Promise<bigint> {
  const publicClient = createPublicClient({
    chain: celo,
    transport: http(),
  });

  const result = await publicClient.simulateContract({
    address: QUOTER_V2,
    abi: QUOTER_ABI,
    functionName: "quoteExactInputSingle",
    args: [
      {
        tokenIn: tokenIn as `0x${string}`,
        tokenOut: tokenOut as `0x${string}`,
        amountIn,
        fee,
        sqrtPriceLimitX96: 0n,
      },
    ],
  });

  return result.result[0];
}
```

## Aave V3 Integration

### Contract Addresses (Mainnet)

| Contract | Address |
|----------|---------|
| Pool | 0x3E59A31363E2ad014dcbc521c4a0d5757d9f3402 |
| PoolAddressesProvider | 0x9F7Cf9417D5251C59fE94fB9147feEe1aAd9Cea5 |
| Oracle | 0x1e693D088ceFD1E95ba4c4a5F7EeA41a1Ec37e8b |
| PoolDataProvider | 0x2e0f8D3B1631296cC7c56538D6Eb6032601E15ED |

### Supported Assets

- CELO
- USDm (cUSD)
- EURm (cEUR)
- USDC
- USDT
- WETH

### Supply Assets

```typescript
const AAVE_POOL = "0x3E59A31363E2ad014dcbc521c4a0d5757d9f3402";

const POOL_ABI = [
  {
    name: "supply",
    type: "function",
    stateMutability: "nonpayable",
    inputs: [
      { name: "asset", type: "address" },
      { name: "amount", type: "uint256" },
      { name: "onBehalfOf", type: "address" },
      { name: "referralCode", type: "uint16" },
    ],
    outputs: [],
  },
] as const;

async function supplyToAave(
  asset: string,
  amount: bigint
): Promise<string> {
  const walletClient = createWalletClient({
    chain: celo,
    transport: custom(window.ethereum),
  });

  const [address] = await walletClient.getAddresses();

  // First approve the Pool to spend tokens
  // ... (see ERC20 approve pattern)

  const hash = await walletClient.writeContract({
    address: AAVE_POOL,
    abi: POOL_ABI,
    functionName: "supply",
    args: [asset as `0x${string}`, amount, address, 0],
  });

  return hash;
}
```

### Borrow Assets

```typescript
const POOL_ABI_BORROW = [
  {
    name: "borrow",
    type: "function",
    stateMutability: "nonpayable",
    inputs: [
      { name: "asset", type: "address" },
      { name: "amount", type: "uint256" },
      { name: "interestRateMode", type: "uint256" },
      { name: "referralCode", type: "uint16" },
      { name: "onBehalfOf", type: "address" },
    ],
    outputs: [],
  },
] as const;

async function borrowFromAave(
  asset: string,
  amount: bigint,
  interestRateMode: number = 2 // 2 = variable rate
): Promise<string> {
  const walletClient = createWalletClient({
    chain: celo,
    transport: custom(window.ethereum),
  });

  const [address] = await walletClient.getAddresses();

  const hash = await walletClient.writeContract({
    address: AAVE_POOL,
    abi: POOL_ABI_BORROW,
    functionName: "borrow",
    args: [asset as `0x${string}`, amount, BigInt(interestRateMode), 0, address],
  });

  return hash;
}
```

## Token Approval

Before interacting with DeFi protocols, approve token spending:

```typescript
const ERC20_ABI = [
  {
    name: "approve",
    type: "function",
    stateMutability: "nonpayable",
    inputs: [
      { name: "spender", type: "address" },
      { name: "amount", type: "uint256" },
    ],
    outputs: [{ type: "bool" }],
  },
] as const;

async function approveToken(
  token: string,
  spender: string,
  amount: bigint
): Promise<string> {
  const walletClient = createWalletClient({
    chain: celo,
    transport: custom(window.ethereum),
  });

  const [address] = await walletClient.getAddresses();

  const hash = await walletClient.writeContract({
    address: token as `0x${string}`,
    abi: ERC20_ABI,
    functionName: "approve",
    args: [spender as `0x${string}`, amount],
  });

  return hash;
}
```

## Fee Tiers

Uniswap V3 uses different fee tiers for pools:

| Fee | Use Case |
|-----|----------|
| 100 (0.01%) | Stable pairs (USDm/USDC) |
| 500 (0.05%) | Stable pairs |
| 3000 (0.3%) | Most pairs |
| 10000 (1%) | Exotic pairs |

## Dependencies

```json
{
  "dependencies": {
    "viem": "^2.0.0"
  }
}
```

For Uniswap SDK:

```json
{
  "dependencies": {
    "@uniswap/v3-sdk": "^3.0.0",
    "@uniswap/sdk-core": "^4.0.0"
  }
}
```

## Additional Resources

- [contract-addresses.md](references/contract-addresses.md) - All DeFi contract addresses
- [uniswap-pools.md](references/uniswap-pools.md) - Popular liquidity pools
