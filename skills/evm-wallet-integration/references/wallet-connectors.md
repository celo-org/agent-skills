# Wallet Connectors Reference

## Wagmi Connectors

Source: https://wagmi.sh/react/guides/connect-wallet

### Injected (Browser Extension)

Connects to any injected wallet (MetaMask, Coinbase, etc.)

```typescript
import { injected } from "wagmi/connectors";

const connector = injected();
```

### MetaMask

Specifically targets MetaMask.

```typescript
import { metaMask } from "wagmi/connectors";

const connector = metaMask();
```

### WalletConnect

Universal wallet connection via QR code or deep link.

```typescript
import { walletConnect } from "wagmi/connectors";

const connector = walletConnect({
  projectId: process.env.NEXT_PUBLIC_WALLETCONNECT_PROJECT_ID!,
});
```

### Coinbase Wallet

Coinbase Wallet and Coinbase Smart Wallet.

```typescript
import { coinbaseWallet } from "wagmi/connectors";

const connector = coinbaseWallet({
  appName: "My Celo App",
});
```

### Safe

Connect to Safe (Gnosis Safe) wallets.

```typescript
import { safe } from "wagmi/connectors";

const connector = safe();
```

## RainbowKit Wallets

Source: https://docs.celo.org/tooling/libraries-sdks/rainbowkit-celo/index.md

### Valora Wallet

Celo's primary mobile wallet.

```typescript
import { valoraWallet } from "@rainbow-me/rainbowkit/wallets";
```

### Celo Wallet

Browser extension wallet.

```typescript
import { celoWallet } from "@rainbow-me/rainbowkit/wallets";
```

### Creating Wallet Groups

```typescript
import { connectorsForWallets } from "@rainbow-me/rainbowkit";
import {
  valoraWallet,
  metaMaskWallet,
  coinbaseWallet,
  walletConnectWallet,
  braveWallet,
  safeWallet,
} from "@rainbow-me/rainbowkit/wallets";

const connectors = connectorsForWallets(
  [
    {
      groupName: "Recommended",
      wallets: [valoraWallet, metaMaskWallet],
    },
    {
      groupName: "Popular",
      wallets: [coinbaseWallet, walletConnectWallet],
    },
    {
      groupName: "Other",
      wallets: [braveWallet, safeWallet],
    },
  ],
  {
    appName: "My Celo App",
    projectId: process.env.NEXT_PUBLIC_WALLETCONNECT_PROJECT_ID!,
  }
);
```

## Complete Wagmi Config with Connectors

```typescript
import { http, createConfig } from "wagmi";
import { celo, celoAlfajores } from "wagmi/chains";
import {
  injected,
  metaMask,
  walletConnect,
  coinbaseWallet,
  safe,
} from "wagmi/connectors";

const projectId = process.env.NEXT_PUBLIC_WALLETCONNECT_PROJECT_ID!;

export const config = createConfig({
  chains: [celo, celoAlfajores],
  connectors: [
    injected(),
    metaMask(),
    walletConnect({ projectId }),
    coinbaseWallet({ appName: "My Celo App" }),
    safe(),
  ],
  transports: {
    [celo.id]: http("https://forno.celo.org"),
    [celoAlfajores.id]: http("https://alfajores-forno.celo-testnet.org"),
  },
});
```

## Thirdweb Wallets

```typescript
import { createWallet } from "thirdweb/wallets";

const wallets = [
  createWallet("io.metamask"),
  createWallet("com.coinbase.wallet"),
  createWallet("app.valora"),
  createWallet("walletConnect"),
];
```

## Wallet Detection

### Check if Wallet Available

```typescript
function isMetaMaskInstalled(): boolean {
  return typeof window !== "undefined" && !!window.ethereum?.isMetaMask;
}

function isWalletConnected(): boolean {
  return typeof window !== "undefined" && !!window.ethereum?.selectedAddress;
}
```

### Get Available Connectors

```tsx
import { useConnectors } from "wagmi";

function AvailableWallets() {
  const connectors = useConnectors();

  return (
    <ul>
      {connectors.map((connector) => (
        <li key={connector.uid}>
          {connector.name} - {connector.type}
        </li>
      ))}
    </ul>
  );
}
```

## Mobile Deep Linking

For mobile wallet connections, ensure proper deep linking:

### Valora

```
celo://wallet/wc?uri=<wc_uri>
```

### MetaMask Mobile

```
https://metamask.app.link/wc?uri=<wc_uri>
```

## WalletConnect Project ID

Required for WalletConnect v2 connections.

1. Go to https://cloud.reown.com
2. Create new project
3. Copy project ID
4. Add to environment variables

```bash
NEXT_PUBLIC_WALLETCONNECT_PROJECT_ID=your_project_id
```
