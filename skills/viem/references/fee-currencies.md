# Fee Currencies Reference

Source: https://viem.sh/docs/chains/celo

## Overview

Celo allows paying gas fees in tokens other than the native CELO currency. The protocol maintains a governable allowlist of tokens through the `FeeCurrencyDirectory` contract.

## Mainnet Fee Currencies

| Token | Token Address | Adapter Address |
|-------|---------------|-----------------|
| USDC | 0xcebA9300f2b948710d2653dD7B07f33A8B32118C | 0x2F25deB3848C207fc8E0c34035B3Ba7fC157602B |
| USDT | 0x48065fbbe25f71c9282ddf5e1cd6d6a887483d5e | 0x0e2a3e05bc9a16f5292a6170456a710cb89c6f72 |

## Celo Sepolia Testnet Fee Currencies

| Token | Token Address | Adapter Address |
|-------|---------------|-----------------|
| USDC | 0x2F25deB3848C207fc8E0c34035B3Ba7fC157602B | 0x4822e58de6f5e485eF90df51C41CE01721331dC0 |

## Why Adapters?

Adapters are needed to normalize decimals for tokens that use different precision. USDC and USDT use 6 decimals, but Celo's gas calculations use 18 decimals. The adapter addresses handle this conversion.

## Usage with Viem

### Serialize Transaction

```typescript
import { serializeTransaction } from "viem/celo";
import { parseGwei, parseEther } from "viem";

// Use the ADAPTER address (not the token address) for feeCurrency
const USDC_ADAPTER = "0x2F25deB3848C207fc8E0c34035B3Ba7fC157602B";

const serialized = serializeTransaction({
  chainId: 42220,
  gas: 21001n,
  feeCurrency: USDC_ADAPTER,
  maxFeePerGas: parseGwei("20"),
  maxPriorityFeePerGas: parseGwei("2"),
  nonce: 69,
  to: "0x1234512345123451234512345123451234512345",
  value: parseEther("0.01"),
});
```

### Send Transaction

```typescript
import { createWalletClient, custom } from "viem";
import { celo } from "viem/chains";

const USDC_ADAPTER = "0x2F25deB3848C207fc8E0c34035B3Ba7fC157602B";

const walletClient = createWalletClient({
  chain: celo,
  transport: custom(window.ethereum),
});

const [address] = await walletClient.getAddresses();

const hash = await walletClient.sendTransaction({
  account: address,
  to: recipient,
  value: 0n,
  feeCurrency: USDC_ADAPTER,
});
```

## Discovering Fee Currencies

### Using celocli

```bash
celocli network:whitelist --node https://forno.celo.org
```

### Using FeeCurrencyDirectory Contract

```typescript
const FEE_CURRENCY_DIRECTORY = "0x9212Fb72ae65367A7c887eC4Ad9bE310BAC611BF";

const currencies = await publicClient.readContract({
  address: FEE_CURRENCY_DIRECTORY,
  abi: [{
    name: "getCurrencies",
    type: "function",
    stateMutability: "view",
    inputs: [],
    outputs: [{ type: "address[]" }],
  }],
  functionName: "getCurrencies",
});
```

## Transaction Type

Fee currency transactions use CIP-64 transaction type `0x7b` (123 in decimal).

## Gas Overhead

Paying the network fee in a stablecoin costs meaningfully more gas than paying in CELO, and **the overhead is not a constant** — it depends on the token, because 6-decimal tokens route through an adapter.

Measured over **2,600 consecutive Celo mainnet blocks** (ending block 77,155,261), comparing `gasUsed` of ERC-20 `transfer` calls paid in CELO against identical transfers that set `feeCurrency`:

| Fee currency | Paid in CELO | Paid in the token | Overhead | Samples (token / CELO) |
|---|---|---|---|---|
| USDm (18-dec, no adapter) | 39,799 | 80,499 | **≈ +40,700** | 1 / 1 |
| USDT (6-dec, adapter) | 43,801 | 115,501 | **≈ +71,700** | 6,970 / 3,143 |
| USDC (6-dec, adapter) | 45,047 | 159,759 | **≈ +114,712** | 7 / 100 |

Medians; distributions are tight (USDT p25 = p75 = 115,501).

**Budget ~2.5–3.5× the gas of an equivalent CELO-paid transaction**, not a flat number — set gas limits from the fee-currency figures or estimation will fail. The USDT row is measured across ~10,000 transactions; USDm and USDC fee-currency samples are small because almost nobody pays fees in them (6,970 of 6,978 fee-currency transfers in the window were USDT), so treat those two rows as indicative.


## Library Support

| Library | feeCurrency Support |
|---------|---------------------|
| viem | ✓ Supported |
| ethers.js | ✗ Not supported |
| web3.js | ✗ Not supported |

Viem is the recommended library for Celo development when using fee currencies.
