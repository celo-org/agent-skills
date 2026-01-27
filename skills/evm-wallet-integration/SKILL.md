---
name: evm-wallet-integration
description: Integrate wallets into Celo dApps. Covers RainbowKit, Dynamic, Reown (WalletConnect), and wallet connection patterns.
license: Apache-2.0
metadata:
  author: celo-org
  version: "1.0.0"
---

# EVM Wallet Integration for Celo

This skill covers integrating wallet connection libraries into Celo dApps.

## When to Use

- Adding wallet connection to a dApp
- Supporting multiple wallet types
- Implementing authentication flows
- Building mobile-friendly wallet experiences

## Wallet Connection Libraries

| Library | Description | Best For |
|---------|-------------|----------|
| RainbowKit | Pre-built UI with wagmi | Production React apps |
| Dynamic | Auth-focused with dashboard | Apps needing user management |
| Reown (WalletConnect) | Universal wallet protocol | Multi-wallet support |
| ConnectKit | Simple wagmi integration | Quick setup |

## RainbowKit

Pre-built, customizable wallet connection for React apps.

Source: https://docs.celo.org/tooling/libraries-sdks/rainbowkit-celo/index.md

### Installation

```bash
npm install @rainbow-me/rainbowkit@2 viem@2 wagmi@2 @tanstack/react-query
```

### Configuration

```typescript
// config.ts
import { getDefaultConfig } from "@rainbow-me/rainbowkit";
import { celo, celoAlfajores } from "wagmi/chains";
import { http } from "wagmi";

export const config = getDefaultConfig({
  appName: "My Celo App",
  projectId: process.env.NEXT_PUBLIC_WALLETCONNECT_PROJECT_ID!,
  chains: [celo, celoAlfajores],
  transports: {
    [celo.id]: http(),
    [celoAlfajores.id]: http(),
  },
});
```

### Provider Setup

```tsx
import "@rainbow-me/rainbowkit/styles.css";
import { RainbowKitProvider } from "@rainbow-me/rainbowkit";
import { WagmiProvider } from "wagmi";
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
import { config } from "./config";

const queryClient = new QueryClient();

function App({ children }: { children: React.ReactNode }) {
  return (
    <WagmiProvider config={config}>
      <QueryClientProvider client={queryClient}>
        <RainbowKitProvider>
          {children}
        </RainbowKitProvider>
      </QueryClientProvider>
    </WagmiProvider>
  );
}
```

### Connect Button

```tsx
import { ConnectButton } from "@rainbow-me/rainbowkit";

function Header() {
  return (
    <nav>
      <ConnectButton />
    </nav>
  );
}
```

### Celo Wallet Support

RainbowKit supports Celo-specific wallets:

```typescript
import { connectorsForWallets } from "@rainbow-me/rainbowkit";
import {
  valoraWallet,
  celoWallet,
  metaMaskWallet,
  coinbaseWallet,
  walletConnectWallet,
} from "@rainbow-me/rainbowkit/wallets";

const connectors = connectorsForWallets(
  [
    {
      groupName: "Recommended",
      wallets: [valoraWallet, celoWallet, metaMaskWallet],
    },
    {
      groupName: "Other",
      wallets: [coinbaseWallet, walletConnectWallet],
    },
  ],
  {
    appName: "My Celo App",
    projectId: process.env.NEXT_PUBLIC_WALLETCONNECT_PROJECT_ID!,
  }
);
```

## Dynamic

Authentication-focused wallet connection with user management dashboard.

Source: https://docs.celo.org/tooling/libraries-sdks/dynamic/index.md

### Installation

```bash
npm install @dynamic-labs/sdk-react
```

### Setup

```tsx
import {
  DynamicContextProvider,
  DynamicWidget,
} from "@dynamic-labs/sdk-react";

function App() {
  return (
    <DynamicContextProvider
      settings={{
        environmentId: process.env.NEXT_PUBLIC_DYNAMIC_ENV_ID!,
      }}
    >
      <DynamicWidget />
    </DynamicContextProvider>
  );
}
```

### Enable Celo

1. Go to app.dynamic.xyz dashboard
2. Navigate to Configurations
3. Select EVM card
4. Toggle Celo on

### Supported Wallets

- Valora
- Celo Wallet
- Coinbase Wallet
- MetaMask
- WalletConnect

## Reown (WalletConnect)

Universal wallet connection protocol.

Source: https://docs.celo.org/tooling/libraries-sdks/reown/index.md

### Get Project ID

1. Go to https://cloud.reown.com
2. Create a new project
3. Copy the project ID

### With Wagmi

```typescript
import { walletConnect } from "wagmi/connectors";

const projectId = process.env.NEXT_PUBLIC_WALLETCONNECT_PROJECT_ID!;

export const config = createConfig({
  chains: [celo],
  connectors: [
    walletConnect({ projectId }),
  ],
  transports: {
    [celo.id]: http(),
  },
});
```

### AppKit (Full Solution)

```bash
npm install @reown/appkit @reown/appkit-adapter-wagmi wagmi viem
```

```tsx
import { createAppKit } from "@reown/appkit/react";
import { WagmiProvider } from "wagmi";
import { celo } from "@reown/appkit/networks";

const projectId = process.env.NEXT_PUBLIC_WALLETCONNECT_PROJECT_ID!;

const metadata = {
  name: "My Celo App",
  description: "My Celo dApp",
  url: "https://myapp.com",
  icons: ["https://myapp.com/icon.png"],
};

createAppKit({
  adapters: [wagmiAdapter],
  networks: [celo],
  metadata,
  projectId,
});
```

## Custom Implementation

Build wallet connection without a library.

### Using Wagmi Directly

```tsx
import { useConnect, useConnectors, useAccount, useDisconnect } from "wagmi";

function WalletConnect() {
  const { connect } = useConnect();
  const connectors = useConnectors();
  const { address, isConnected } = useAccount();
  const { disconnect } = useDisconnect();

  if (isConnected) {
    return (
      <div>
        <p>Connected: {address}</p>
        <button onClick={() => disconnect()}>Disconnect</button>
      </div>
    );
  }

  return (
    <div>
      {connectors.map((connector) => (
        <button
          key={connector.uid}
          onClick={() => connect({ connector })}
        >
          {connector.name}
        </button>
      ))}
    </div>
  );
}
```

## Network Configuration

### Celo Networks

| Network | Chain ID | Import |
|---------|----------|--------|
| Mainnet | 42220 | `celo` |
| Alfajores | 44787 | `celoAlfajores` |
| Celo Sepolia | 11142220 | Custom |

### Celo Sepolia (Custom)

```typescript
import { defineChain } from "viem";

export const celoSepolia = defineChain({
  id: 11142220,
  name: "Celo Sepolia",
  nativeCurrency: { name: "CELO", symbol: "CELO", decimals: 18 },
  rpcUrls: {
    default: { http: ["https://forno.celo-sepolia.celo-testnet.org"] },
  },
  blockExplorers: {
    default: {
      name: "Celo Sepolia Explorer",
      url: "https://celo-sepolia.blockscout.com",
    },
  },
  testnet: true,
});
```

## Mobile Wallet Support

### Valora

Valora is the primary mobile wallet for Celo. Ensure your dApp:

1. Uses WalletConnect for mobile connections
2. Has proper deep linking configured
3. Tests on mobile devices

### Mobile Detection

```typescript
function isMobile(): boolean {
  return /Android|iPhone|iPad|iPod/i.test(navigator.userAgent);
}
```

## Best Practices

1. **Support Multiple Wallets** - Don't force users into one wallet
2. **Handle Network Switching** - Prompt users to switch to Celo
3. **Show Connection State** - Clear UI for connected/disconnected
4. **Handle Errors** - User-friendly error messages
5. **Test on Mobile** - Valora and mobile browsers

## Dependencies

```json
{
  "dependencies": {
    "@rainbow-me/rainbowkit": "^2.0.0",
    "wagmi": "^2.0.0",
    "viem": "^2.0.0",
    "@tanstack/react-query": "^5.0.0"
  }
}
```

## Additional Resources

- [wallet-connectors.md](references/wallet-connectors.md) - Connector configuration reference
