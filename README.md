# Celo Agent Skills

Agent Skills for building on Celo and EVM chains. Compatible with [Claude Code](https://claude.ai/code), [Cursor](https://cursor.com), [Windsurf](https://codeium.com/windsurf), [OpenAI Codex](https://openai.com/codex), and other [Agent Skills](https://agentskills.io) compatible tools.

## Installation

```bash
# Install all skills
npx openskills install celo-org/agent-skills -g

# Install a specific skill
npx openskills install celo-org/agent-skills --skill evm-hardhat -g
```

### Package managers

```bash
# pnpm
pnpm dlx openskills install celo-org/agent-skills -g

# yarn
yarn dlx openskills install celo-org/agent-skills -g

# bun
bunx openskills install celo-org/agent-skills -g
```

## Skills

| Skill | Description |
|-------|-------------|
| [evm-hardhat](skills/evm-hardhat) | Hardhat development for EVM chains including Celo. Covers project setup, compilation, testing, deployment, and verification. |
| [evm-foundry](skills/evm-foundry) | Foundry development for EVM chains including Celo. Covers forge, cast, anvil, testing, deployment, and verification. |
| [celo-rpc](skills/celo-rpc) | Interact with Celo blockchain via RPC. Covers reading balances, transactions, blocks, and Celo-specific methods like fee currency. |

## Skill Structure

Each skill follows the [Agent Skills](https://agentskills.io) specification:

```
skill-name/
├── SKILL.md           # Main instructions (required)
├── references/        # Detailed documentation
├── rules/             # Best practices and standards
└── scripts/           # Executable scripts
```

Skills activate automatically when relevant tasks are detected.

## Celo Network Information

| Network | Chain ID | RPC Endpoint |
|---------|----------|--------------|
| Celo Mainnet | 42220 | https://forno.celo.org |
| Celo Sepolia | 11142220 | https://forno.celo-sepolia.celo-testnet.org |


## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

## License

Apache-2.0
