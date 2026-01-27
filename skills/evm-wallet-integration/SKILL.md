---
name: evm-wallet-integration
description: Integrate wallets into Celo dApps. Covers RainbowKit, Dynamic, and wallet connection patterns.
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
- Building wallet experiences

## Wallet Connection Libraries

| Library | Description | Best For |
|---------|-------------|----------|
| RainbowKit | Pre-built UI with wagmi | Production React apps |
| Dynamic | Auth-focused with dashboard | Apps needing user management |
| ConnectKit | Simple wagmi integration | Quick setup |

## RainbowKit

Pre-built, customizable wallet connection for React apps.

Source: https://www.rainbowkit.com

### Installation

```bash
npm install @rainbow-me/rainbowkit@2 viem@2 wagmi@2 @tanstack/react-query
```

### Configuration

```typescript
// config.ts
import { getDefaultConfig } from "@rainbow-me/rainbowkit";
import { celo, celoSepolia } from "wagmi/chains";
import { http } from "wagmi";

export const config = getDefaultConfig({
  appName: "My Celo App",
  projectId: process.env.NEXT_PUBLIC_WALLETCONNECT_PROJECT_ID!,
  chains: [celo, celoSepolia],
  transports: {
    [celo.id]: http(),
    [celoSepolia.id]: http(),
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

### Custom Wallet Groups

```typescript
import { connectorsForWallets } from "@rainbow-me/rainbowkit";
import {
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
      wallets: [metaMaskWallet, coinbaseWallet],
    },
    {
      groupName: "Other",
      wallets: [walletConnectWallet, braveWallet, safeWallet],
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

Source: https://docs.dynamic.xyz

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

## Custom Implementation

Build wallet connection without a library using wagmi directly.

### Wagmi Configuration

```typescript
import { http, createConfig } from "wagmi";
import { celo, celoSepolia } from "wagmi/chains";
import { injected, walletConnect, metaMask } from "wagmi/connectors";

const projectId = process.env.NEXT_PUBLIC_WALLETCONNECT_PROJECT_ID!;

export const config = createConfig({
  chains: [celo, celoSepolia],
  connectors: [
    injected(),
    walletConnect({ projectId }),
    metaMask(),
  ],
  transports: {
    [celo.id]: http(),
    [celoSepolia.id]: http(),
  },
});
```

### Wallet Connect Component

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
| Celo Sepolia | 11142220 | `celoSepolia` |

### WalletConnect Project ID

Required for WalletConnect connections.

1. Go to https://cloud.reown.com
2. Create new project
3. Copy project ID
4. Add to environment variables

```bash
NEXT_PUBLIC_WALLETCONNECT_PROJECT_ID=your_project_id
```

## Best Practices

1. **Support Multiple Wallets** - Don't force users into one wallet
2. **Handle Network Switching** - Prompt users to switch to Celo
3. **Show Connection State** - Clear UI for connected/disconnected
4. **Handle Errors** - User-friendly error messages
5. **Test on Mobile** - Mobile browsers and wallet apps

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
