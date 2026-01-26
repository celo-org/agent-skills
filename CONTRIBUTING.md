# Contributing to Celo Agent Skills

Thank you for your interest in contributing to Celo Agent Skills!

## Getting Started

1. Fork the repository
2. Clone your fork locally
3. Create a new branch for your changes

## Skill Structure

Each skill follows the [Agent Skills](https://agentskills.io) specification:

```
skill-name/
├── SKILL.md           # Main instructions (required)
├── references/        # Detailed documentation
├── rules/             # Best practices and standards
└── scripts/           # Executable scripts
```

### SKILL.md Format

```yaml
---
name: skill-name
description: What the skill does and when to use it.
license: Apache-2.0
metadata:
  author: celo-org
  version: "1.0.0"
---

# Skill Title

Content goes here...
```

## Quality Standards

### Before Submitting

- [ ] SKILL.md follows the Agent Skills spec
- [ ] All contract addresses are verified on block explorers
- [ ] Code examples have been tested
- [ ] No deprecated APIs or testnets
- [ ] References link to current documentation
- [ ] All links are valid (no 404s)

### Code Examples

- All code examples must compile
- Use the latest stable Solidity version
- Test on Celo Sepolia before documenting

### Documentation

- Keep SKILL.md under 500 lines
- Move detailed reference material to separate files
- Use consistent formatting

## Pull Request Process

1. Update the README.md if adding a new skill
2. Ensure all tests pass
3. Update version numbers if applicable
4. Request review from maintainers

## Code of Conduct

Please be respectful and constructive in all interactions.

## Questions?

Open an issue for questions or discussions.
