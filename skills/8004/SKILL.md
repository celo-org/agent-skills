---
name: 8004
description: ERC-8004 Agent Trust Protocol for AI agent identity, reputation, and validation on Celo. Use when building AI agents that need identity registration, reputation tracking, or trust verification across organizational boundaries.
license: Apache-2.0
metadata:
  author: celo-org
  version: "1.0.0"
---

# ERC-8004: Agent Trust Protocol

ERC-8004 establishes trust infrastructure for autonomous AI agents, enabling them to discover, identify, and evaluate other agents across organizational boundaries.

## When to Use

- Registering an AI agent's identity on-chain
- Building reputation systems for AI agents
- Verifying agent identity before interactions
- Querying agent reputation and feedback
- Implementing trust-based agent interactions

## Core Concepts

### The Three Registries

| Registry | Purpose | Key Functions |
|----------|---------|---------------|
| **Identity Registry** | Agent discovery via ERC-721 NFTs | `register()`, `tokenURI()`, `getAgentWallet()`, `setMetadata()`, `getMetadata()` |
| **Reputation Registry** | Feedback and attestations | `giveFeedback()`, `revokeFeedback()`, `readFeedback()`, `readAllFeedback()`, `getSummary()`, `appendResponse()`, `getClients()` |
| **Validation Registry** | Verification hooks | Custom validators |

### Protocol Stack Position

```
Application Layer (Agent Apps, Marketplaces)
    ↓
Trust Layer (ERC-8004) ← This skill
    ↓
Payment Layer (x402)
    ↓
Communication Layer (A2A, MCP)
```

## Installation

```bash
npm install viem
```

> ABIs are available at `skills/8004/references/identity-registry-abi.json` and `skills/8004/references/reputation-registry-abi.json`.

## Contract Addresses

### Celo Mainnet (Chain ID: 42220)

| Contract | Address |
|----------|---------|
| Identity Registry | `0x8004A169FB4a3325136EB29fA0ceB6D2e539a432` |
| Reputation Registry | `0x8004BAa17C55a88189AE136b182e5fdA19dE9b63` |

### Celo Sepolia Testnet (Chain ID: 11142220)

| Contract | Address |
|----------|---------|
| Identity Registry | `0x8004A818BFB912233c491871b3d84c89A494BD9e` |
| Reputation Registry | `0x8004B663056A597Dffe9eCcC1965A193B7388713` |

## Setup (viem)

```javascript
import { createPublicClient, createWalletClient, http, getContract } from 'viem';
import { privateKeyToAccount } from 'viem/accounts';
import { celo, celoAlfajores } from 'viem/chains';
import identityRegistryAbi from './references/identity-registry-abi.json';
import reputationRegistryAbi from './references/reputation-registry-abi.json';

// Celo Mainnet addresses
const IDENTITY_REGISTRY = '0x8004A169FB4a3325136EB29fA0ceB6D2e539a432';
const REPUTATION_REGISTRY = '0x8004BAa17C55a88189AE136b182e5fdA19dE9b63';

// Celo Sepolia Testnet addresses
// const IDENTITY_REGISTRY = '0x8004A818BFB912233c491871b3d84c89A494BD9e';
// const REPUTATION_REGISTRY = '0x8004B663056A597Dffe9eCcC1965A193B7388713';

const account = privateKeyToAccount('0xYOUR_PRIVATE_KEY');

const publicClient = createPublicClient({
  chain: celo,
  transport: http('https://forno.celo.org'),
});

const walletClient = createWalletClient({
  account,
  chain: celo,
  transport: http('https://forno.celo.org'),
});
```

## Agent Registration

### 1. Agent Metadata Format

Create an agent metadata JSON and host it (IPFS, HTTPS, etc.):

```json
{
  "type": "Agent",
  "name": "My AI Agent",
  "description": "Description of capabilities",
  "image": "ipfs://Qm...",
  "endpoints": [
    { "type": "a2a", "url": "https://example.com/.well-known/agent.json" },
    { "type": "mcp", "url": "https://example.com/mcp" },
    { "type": "wallet", "address": "0x...", "chainId": 42220 }
  ],
  "supportedTrust": ["reputation", "validation", "tee"]
}
```

### 2. Register Agent On-Chain

```javascript
// Register without URI (minimal)
const hash = await walletClient.writeContract({
  address: IDENTITY_REGISTRY,
  abi: identityRegistryAbi,
  functionName: 'register',
  args: [],
});

// Register with metadata URI
const hash = await walletClient.writeContract({
  address: IDENTITY_REGISTRY,
  abi: identityRegistryAbi,
  functionName: 'register',
  args: ['ipfs://QmYourAgentMetadata'],
});

// Register with URI + custom metadata key-value pairs
const hash = await walletClient.writeContract({
  address: IDENTITY_REGISTRY,
  abi: identityRegistryAbi,
  functionName: 'register',
  args: [
    'ipfs://QmYourAgentMetadata',
    [
      { metadataKey: 'category', metadataValue: '0x' + Buffer.from('defi-trading').toString('hex') },
      { metadataKey: 'version', metadataValue: '0x' + Buffer.from('1.0.0').toString('hex') },
    ],
  ],
});

// Get agent ID from transaction receipt
const receipt = await publicClient.waitForTransactionReceipt({ hash });
const transferLog = receipt.logs.find(log => log.topics.length === 4); // Transfer event
const agentId = BigInt(transferLog.topics[3]); // tokenId is the 3rd indexed param
console.log('Agent registered with ID:', agentId);
```

### 3. Read Agent Data

```javascript
// Get agent metadata URI
const agentURI = await publicClient.readContract({
  address: IDENTITY_REGISTRY,
  abi: identityRegistryAbi,
  functionName: 'tokenURI',
  args: [agentId],
});

// Get agent wallet address
const agentWallet = await publicClient.readContract({
  address: IDENTITY_REGISTRY,
  abi: identityRegistryAbi,
  functionName: 'getAgentWallet',
  args: [agentId],
});

// Get agent owner (NFT holder)
const owner = await publicClient.readContract({
  address: IDENTITY_REGISTRY,
  abi: identityRegistryAbi,
  functionName: 'ownerOf',
  args: [agentId],
});

// Get custom metadata
const categoryBytes = await publicClient.readContract({
  address: IDENTITY_REGISTRY,
  abi: identityRegistryAbi,
  functionName: 'getMetadata',
  args: [agentId, 'category'],
});
```

### 4. Update Agent

```javascript
// Update agent URI
await walletClient.writeContract({
  address: IDENTITY_REGISTRY,
  abi: identityRegistryAbi,
  functionName: 'setAgentURI',
  args: [agentId, 'ipfs://QmUpdatedMetadata'],
});

// Set custom metadata
await walletClient.writeContract({
  address: IDENTITY_REGISTRY,
  abi: identityRegistryAbi,
  functionName: 'setMetadata',
  args: [agentId, 'status', '0x' + Buffer.from('active').toString('hex')],
});
```

## Reputation System

### Give Feedback

> **Note:** Self-feedback is blocked — the agent owner/operators cannot give feedback to their own agent. The `value` field is `int128` (supports negative scores). `valueDecimals` max is 18.

```javascript
import { keccak256, toBytes } from 'viem';

const feedbackContent = JSON.stringify({ quality: 'excellent', notes: 'Fast response' });
const feedbackHash = keccak256(toBytes(feedbackContent));

await walletClient.writeContract({
  address: REPUTATION_REGISTRY,
  abi: reputationRegistryAbi,
  functionName: 'giveFeedback',
  args: [
    agentId,                              // agentId (uint256)
    85n,                                  // value (int128) — can be negative
    0,                                    // valueDecimals (uint8)
    'starred',                            // tag1: category
    '',                                   // tag2: optional sub-category
    'https://agent.example.com',          // endpoint used
    'ipfs://QmDetailedFeedback',          // feedbackURI for details
    feedbackHash,                         // keccak256 of feedback content
  ],
});
```

### Common Feedback Tags

| Tag | Measures | Example |
|-----|----------|---------|
| `starred` | Quality rating (0-100) | 87/100 |
| `uptime` | Endpoint uptime % | 99.77% |
| `successRate` | Task success rate % | 89% |
| `responseTime` | Response time (ms) | 560ms |
| `reachable` | Endpoint reachable | 1/0 |

### Revoke Feedback

```javascript
// Revoke feedback at a specific index (1-indexed)
await walletClient.writeContract({
  address: REPUTATION_REGISTRY,
  abi: reputationRegistryAbi,
  functionName: 'revokeFeedback',
  args: [agentId, 1n], // feedbackIndex (uint64, 1-indexed)
});
```

### Respond to Feedback

Agents (or anyone) can append responses to feedback entries:

```javascript
await walletClient.writeContract({
  address: REPUTATION_REGISTRY,
  abi: reputationRegistryAbi,
  functionName: 'appendResponse',
  args: [
    agentId,
    clientAddress,        // address of feedback author
    1n,                   // feedbackIndex
    'ipfs://QmResponse',  // responseURI
    responseHash,         // keccak256 of response content
  ],
});
```

### Query Reputation

```javascript
// Get all clients who gave feedback
const clients = await publicClient.readContract({
  address: REPUTATION_REGISTRY,
  abi: reputationRegistryAbi,
  functionName: 'getClients',
  args: [agentId],
});

// Read a single feedback entry
const [value, valueDecimals, tag1, tag2, isRevoked] = await publicClient.readContract({
  address: REPUTATION_REGISTRY,
  abi: reputationRegistryAbi,
  functionName: 'readFeedback',
  args: [agentId, clientAddress, 1n], // feedbackIndex (1-indexed)
});

// Read all feedback (pass clients array, filter by tags)
const allFeedback = await publicClient.readContract({
  address: REPUTATION_REGISTRY,
  abi: reputationRegistryAbi,
  functionName: 'readAllFeedback',
  args: [
    agentId,
    clients,    // clientAddresses — pass [] to auto-use all known clients
    '',         // tag1 filter ('' = all)
    '',         // tag2 filter ('' = all)
    false,      // includeRevoked
  ],
});
// Returns: { clients, feedbackIndexes, values, valueDecimals, tag1s, tag2s, revokedStatuses }

// Get aggregated summary (requires clientAddresses)
const [count, summaryValue, summaryValueDecimals] = await publicClient.readContract({
  address: REPUTATION_REGISTRY,
  abi: reputationRegistryAbi,
  functionName: 'getSummary',
  args: [
    agentId,
    clients,  // clientAddresses (required, cannot be empty)
    '',       // tag1 filter
    '',       // tag2 filter
  ],
});
console.log(`Average: ${summaryValue} (${count} reviews, ${summaryValueDecimals} decimals)`);
```

## Trust Verification Workflow

```javascript
async function verifyAndInteract(targetAgentId, minScore = 70n) {
  // 1. Verify identity exists (reverts if not)
  const owner = await publicClient.readContract({
    address: IDENTITY_REGISTRY,
    abi: identityRegistryAbi,
    functionName: 'ownerOf',
    args: [targetAgentId],
  });

  // 2. Get clients and check reputation
  const clients = await publicClient.readContract({
    address: REPUTATION_REGISTRY,
    abi: reputationRegistryAbi,
    functionName: 'getClients',
    args: [targetAgentId],
  });

  if (clients.length > 0) {
    const [count, summaryValue] = await publicClient.readContract({
      address: REPUTATION_REGISTRY,
      abi: reputationRegistryAbi,
      functionName: 'getSummary',
      args: [targetAgentId, clients, '', ''],
    });
    if (summaryValue < minScore) {
      throw new Error(`Agent reputation ${summaryValue} below threshold ${minScore}`);
    }
  }

  // 3. Fetch agent metadata and find endpoint
  const agentURI = await publicClient.readContract({
    address: IDENTITY_REGISTRY,
    abi: identityRegistryAbi,
    functionName: 'tokenURI',
    args: [targetAgentId],
  });
  const agentData = await fetch(agentURI).then(r => r.json());
  const endpoint = agentData.endpoints.find(e => e.type === 'a2a');

  // 4. Interact with verified agent
  const result = await interactWithAgent(endpoint.url);

  // 5. Submit feedback
  await walletClient.writeContract({
    address: REPUTATION_REGISTRY,
    abi: reputationRegistryAbi,
    functionName: 'giveFeedback',
    args: [
      targetAgentId,
      result.success ? 90n : 30n,
      0,
      result.success ? 'starred' : 'failed',
      '',
      endpoint.url,
      '',
      '0x0000000000000000000000000000000000000000000000000000000000000000',
    ],
  });

  return result;
}
```

## Integration with x402 Payments

ERC-8004 and x402 work together for trustworthy paid agent interactions:

```javascript
import { wrapFetchWithPayment } from 'thirdweb/x402';

async function payTrustedAgent(agentId, serviceUrl) {
  // 1. Verify trust via reputation
  const clients = await publicClient.readContract({
    address: REPUTATION_REGISTRY,
    abi: reputationRegistryAbi,
    functionName: 'getClients',
    args: [agentId],
  });

  if (clients.length > 0) {
    const [count, summaryValue] = await publicClient.readContract({
      address: REPUTATION_REGISTRY,
      abi: reputationRegistryAbi,
      functionName: 'getSummary',
      args: [agentId, clients, '', ''],
    });
    if (summaryValue < 80n) {
      throw new Error('Agent not trusted enough for payment');
    }
  }

  // 2. Make paid request via x402
  const fetchWithPayment = wrapFetchWithPayment({
    client,
    account,
    paymentOptions: { maxValue: "1000000" },
  });

  const response = await fetchWithPayment(serviceUrl);
  return response.json();
}
```

## Validation Registry

For high-stakes operations, use Validation Registry for additional verification:

| Model | Mechanism | Best For |
|-------|-----------|----------|
| **Reputation-based** | Client feedback | Low-stake, frequent |
| **Crypto-economic** | Stake + slashing | Medium-stake financial |
| **zkML** | Zero-knowledge proofs | Privacy-preserving |
| **TEE Attestation** | Hardware isolation | High-assurance |

## Celo Network Reference

| Network | Chain ID | RPC Endpoint |
|---------|----------|--------------|
| Celo Mainnet | 42220 | https://forno.celo.org |
| Celo Sepolia | 11142220 | https://forno.celo-sepolia.celo-testnet.org |

## Additional Resources

- [EIP Specification](https://eips.ethereum.org/EIPS/eip-8004)
- [Official Website](https://www.8004.org)
- [Learning Portal](https://www.8004.org/learn)
- [Contracts Repository](https://github.com/erc-8004/erc-8004-contracts)
- [Builder Program](http://bit.ly/8004builderprogram)

## Related Skills

- [x402](../x402/SKILL.md) - Payment layer for AI agents
- [celo-rpc](../celo-rpc/SKILL.md) - Celo blockchain interaction
- [viem](../viem/SKILL.md) - TypeScript Ethereum library
