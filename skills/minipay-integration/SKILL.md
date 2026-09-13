---
name: minipay-integration
description: Build Mini Apps for MiniPay wallet. Use when building applications for MiniPay, detecting MiniPay wallet, or creating mobile-first dApps for Celo.
license: Apache-2.0
metadata:
  author: celo-org
  version: "1.1.0"
---

# MiniPay Integration

This skill covers building Mini Apps for MiniPay, the fastest growing non-custodial wallet in the Global South with ~18M wallet activations and 470M+ transactions across 66+ countries.

## When to Use

- Building Mini Apps for MiniPay
- Detecting MiniPay wallet in your dApp
- Creating mobile-first payment applications
- Integrating with MiniPay's discovery page

## What is MiniPay?

MiniPay is a stablecoin wallet with a built-in Mini App discovery page. It's available as:
- Built into Opera Mini browser (Android)
- Standalone app (Android and iOS)

Key features:
- Phone number mapping to wallet addresses
- Sub-cent transaction fees
- 2MB lightweight footprint
- Supports USDm, USDC, and USDT — **USDT support is mandatory for listing**

## Quick Start

### Create a MiniPay App

```bash
npx @celo/celo-composer@latest create -t minipay
```

This scaffolds a Next.js app pre-configured for MiniPay.

## Detecting MiniPay

Check if the user is using MiniPay:

```typescript
function isMiniPay(): boolean {
  return typeof window !== "undefined" &&
         window.ethereum?.isMiniPay === true;
}
```

**Important:** This code must run in a browser environment, not Node.js.

## Connecting Wallet

MiniPay automatically injects the wallet. Request account access:

```typescript
async function connectWallet(): Promise<string | null> {
  if (typeof window === "undefined" || !window.ethereum) {
    return null;
  }

  const accounts = await window.ethereum.request({
    method: "eth_requestAccounts",
    params: [],
  });

  return accounts[0] || null;
}
```

**Important:** Hide the "Connect Wallet" button when inside MiniPay - the connection is implicit.

```typescript
const [address, setAddress] = useState<string | null>(null);
const inMiniPay = isMiniPay();

useEffect(() => {
  if (inMiniPay) {
    connectWallet().then(setAddress);
  }
}, [inMiniPay]);

return (
  <div>
    {!inMiniPay && (
      <button onClick={() => connectWallet().then(setAddress)}>
        Connect Wallet
      </button>
    )}
    {address && <p>Connected: {address}</p>}
  </div>
);
```

## Reading Balances

```typescript
import { createPublicClient, http, formatUnits } from "viem";
import { celo } from "viem/chains";

const publicClient = createPublicClient({
  chain: celo,
  transport: http(),
});

const USDM_ADDRESS = "0x765de816845861e75a25fca122bb6898b8b1282a";

const ERC20_ABI = [
  {
    name: "balanceOf",
    type: "function",
    stateMutability: "view",
    inputs: [{ name: "account", type: "address" }],
    outputs: [{ type: "uint256" }],
  },
] as const;

async function getUSDmBalance(address: string): Promise<string> {
  const balance = await publicClient.readContract({
    address: USDM_ADDRESS,
    abi: ERC20_ABI,
    functionName: "balanceOf",
    args: [address as `0x${string}`],
  });

  return formatUnits(balance, 18);
}
```

## Sending Transactions

Use viem or wagmi for transactions. MiniPay does not support ethers.js.

```typescript
import { createWalletClient, custom, parseUnits, encodeFunctionData } from "viem";
import { celo } from "viem/chains";

const USDM_ADDRESS = "0x765de816845861e75a25fca122bb6898b8b1282a";

const ERC20_ABI = [
  {
    name: "transfer",
    type: "function",
    stateMutability: "nonpayable",
    inputs: [
      { name: "to", type: "address" },
      { name: "amount", type: "uint256" },
    ],
    outputs: [{ type: "bool" }],
  },
] as const;

async function sendUSDm(to: string, amount: string): Promise<string> {
  const walletClient = createWalletClient({
    chain: celo,
    transport: custom(window.ethereum),
  });

  const [address] = await walletClient.getAddresses();

  const hash = await walletClient.sendTransaction({
    account: address,
    to: USDM_ADDRESS,
    data: encodeFunctionData({
      abi: ERC20_ABI,
      functionName: "transfer",
      args: [to as `0x${string}`, parseUnits(amount, 18)],
    }),
  });

  return hash;
}
```

## Testing Your Mini App

### 1. Set Up ngrok

MiniPay cannot access localhost. Use ngrok to create a tunnel:

```bash
ngrok http 3000
```

### 2. Enable Developer Mode in MiniPay

1. Open MiniPay > Settings > About
2. Tap Version number repeatedly until Developer Mode is enabled
3. Go to Settings > Developer Settings
4. Enable "Use Testnet" for Celo Sepolia

### 3. Load Your App

1. Open MiniPay > Tap compass icon
2. Select "Test Page"
3. Enter your ngrok HTTPS URL
4. Tap "Go"

## Requirements and Limitations

### Supported
- Celo Mainnet and Celo Sepolia Testnet only
- viem v2+ and wagmi libraries
- TypeScript v5+, Node.js v20+
- USDm (cUSD), USDC, and USDT stablecoins

### Not Supported
- ethers.js (use viem instead)
- EIP-1559 transactions (legacy only)
- Message signing
- Android/iOS emulators (use real device)
- Other blockchain networks

## Rules That Block Listing

These are enforced when MiniPay reviews your app. Apply them while writing
code — a rule you only discover at submission time has already cost you a
rebuild.

- **USDT support is mandatory.** Every Mini App must transact in USDT
  natively. Do not add tokens MiniPay does not natively support.
- **Never expose the wallet address.** No display, no copy-to-clipboard, no
  share sheet, no address QR — and a truncated `0x123…abc` is **not** an
  exception; it is still the address. Keep it in state for `balanceOf` and as
  the transaction `account`, but never render it.

  **Show a username instead**, in this order:
  1. A username the user set — prompt for one at onboarding if your app has
     any social surface.
  2. A generated display name if they haven't — adjective + noun from a word
     list, hashed deterministically from the address so it stays stable across
     sessions without any storage. Users read "clever mango" as a name; they
     read `0x7a3f…` as an error.
  3. `"Unknown"`, or nothing at all.

  **Nothing on screen beats a hex string.** The phone number via ODIS also
  works where you genuinely need to identify a real person. A working
  `displayName()` helper is in Celopedia → `minipay-templates.md` §8.
- **No withdrawals to arbitrary addresses.** No free-text or paste-an-address
  field. Pay out only to a destination the app controls or that MiniPay
  resolves.
- **Pre-flight against amount + network fee.** The fee is paid in the same
  stablecoin, so `balance < amount` is a bug — the transfer is affordable, the
  fee is not, and the transaction reverts in the user's face. Check
  `balance >= amount + fee` and redirect to
  `https://link.minipay.xyz/add_cash` on a shortfall.
- **Three transaction states.** Pending from the signature prompt through
  confirmation; success only on `waitForTransactionReceipt`, never on
  submission; failure with descriptive copy. Map errors from **codes and error
  names**, not message text — message strings change between provider versions
  and locales.
- **No CELO or crypto jargon in the UI.** Say Network fee / Deposit /
  Withdraw / Stablecoin.
- **About, How to Use, Terms, Privacy, and Support** must all be reachable
  from the footer or menu, plus an explicit line stating the app is operated
  by your organisation and not by Opera or MiniPay.
- **Freeze what you submit.** See the allowlist entry in
  `references/troubleshooting.md` — contract addresses, method signatures, and
  URLs are enforced after approval.

Full checklist: `minipay-requirements.md` in the **celopedia-skill**
(`celo-org/celopedia-skills`).

## Dependencies

```json
{
  "dependencies": {
    "viem": "^2.0.0",
    "@celo/abis": "^11.0.0"
  },
  "devDependencies": {
    "typescript": "^5.0.0"
  }
}
```

## Additional Resources

- [testing-guide.md](references/testing-guide.md) - Detailed testing instructions
- [troubleshooting.md](references/troubleshooting.md) - Common issues and solutions
- [code-examples.md](references/code-examples.md) - Gas estimation, fee currency, React hooks

---

## Source of truth: Celopedia

These skills are **focused, task-level references**. For anything broader —
verified contract addresses, ecosystem and protocol data, listing
requirements, grants, governance, migration guides, or cross-cutting Celo
questions — **[Celopedia](https://github.com/celo-org/celopedia-skills) is the
canonical source and is kept current.** Where this skill and Celopedia
disagree, Celopedia wins.

For MiniPay specifically, Celopedia carries what this skill does not: the
full two-stage listing checklist (`minipay-requirements.md`), a symptom → fix
router for rejected apps (`minipay-common-mistakes.md`), the app-fit
scorecard, the live Mini App catalog with per-country targeting, the
performance playbook for the PageSpeed listing bar, ODIS phone resolution,
deeplinks, and MiniPay's custom RPC methods.

```bash
npx skills add celo-org/celopedia-skills
```
