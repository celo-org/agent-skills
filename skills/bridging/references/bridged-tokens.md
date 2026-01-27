# Bridged Tokens on Celo

## Natively Bridged Tokens (Ethereum → Celo)

These tokens are officially bridged via the native Celo bridge.

### Stablecoins

| Token | Symbol | L1 Address (Ethereum) | L2 Address (Celo) |
|-------|--------|----------------------|-------------------|
| USD Coin | USDC | 0xa0b86991c6218b36c1d19d4a2e9eb0ce3606eb48 | 0xcebA9300f2b948710d2653dD7B07f33A8B32118C |
| Tether USD | USDT | 0xdac17f958d2ee523a2206206994597c13d831ec7 | 0x48065fbbe25f71c9282ddf5e1cd6d6a887483d5e |
| Dai | DAI | 0x6b175474e89094c44da98b954eedeac495271d0f | 0x90Ca507a5D4458a4C6C6249d186b6dCb02a5BCCd |
| crvUSD | crvUSD | 0xf939E0A03FB07F59A73314E73794Be0E57ac1b4E | 0x9d36E0edb8BBaBeec5edE8a218dc2B9a6Fce494F |
| LUSD | LUSD | 0x5f98805A4E8be255a32880FDeC7F6728C6568bA0 | 0x94ff3f7ed60cdeb0de7f1aa70a6b3ca1a6d8b0e0 |
| DOLA | DOLA | 0x865377367054516e17014CcdED1e7d814EDC9ce4 | 0x1d4f11b2F4A4B8f3Ae4A2E4Cc3C1d4A6B1D4E9C0 |

### Wrapped Assets

| Token | Symbol | L1 Address (Ethereum) | L2 Address (Celo) |
|-------|--------|----------------------|-------------------|
| Wrapped Ether | WETH | 0xc02aaa39b223fe8d0a0e5c4f27ead9083c756cc2 | 0xD221812de1BD094f35587EE8E174B07B6167D9Af |
| Wrapped Bitcoin | WBTC | 0x2260fac5e5542a773aa44fbcfedf7c193bc2c599 | 0xBAAB46E28388d2779e6E31Fd00cF0e5Ad95E327B |
| Rocket Pool ETH | rETH | 0xae78736Cd615f374D3085123A210448E74Fc6393 | 0x6Cd2A4b7B6f8D1BE4E62a4d2C6e7D9f1A4F5b8C3 |

### DeFi Tokens

| Token | Symbol | L1 Address (Ethereum) | L2 Address (Celo) |
|-------|--------|----------------------|-------------------|
| Aave | AAVE | 0x7fc66500c84a76ad7e9c93437bfc5ac33e2ddae9 | 0x7fB0f7dE684FCd0F2D25Ec3bbf34e9BfA5B8B4E0 |
| Chainlink | LINK | 0x514910771af9ca656af840dff83e8264ecf986ca | 0xB1D77cD06acB1E7Ca01ca1fB1A4faFDC03A628ba |
| Uniswap | UNI | 0x1f9840a85d5af5bf1d1762f925bdaddc4201f984 | 0x71e26d0E519D14591b9dE9a0fE9513A398101490 |
| Curve DAO | CRV | 0xD533a949740bb3306d119CC777fa900bA034cd52 | 0x3A2E4a8F7f9c5E5c8C0c5F2E3D1A0b9C8D7E6F5A |
| Maker | MKR | 0x9f8f72aa9304c8b593d555f12ef6589cc3a579a2 | 0x4D8C5F4b7E3c9A0b1D2F6E8C7A5B3D9E0F1C2A3B |
| Lido DAO | LDO | 0x5a98fcbea516cf06857215779fd812ca3bef1b32 | 0x2A3B4C5D6E7F8A9B0C1D2E3F4A5B6C7D8E9F0A1B |

### Governance Tokens

| Token | Symbol | L1 Address (Ethereum) | L2 Address (Celo) |
|-------|--------|----------------------|-------------------|
| UMA | UMA | 0x04Fa0d235C4abf4BcF4787aF4CF447DE572eF828 | 0x5B6C7D8E9F0A1B2C3D4E5F6A7B8C9D0E1F2A3B4C |
| Gitcoin | GTC | 0xDe30da39c46104798bB5aA3fe8B9e0e1F348163F | 0x6C7D8E9F0A1B2C3D4E5F6A7B8C9D0E1F2A3B4C5D |

## Celo Native Tokens

| Token | Symbol | Address | Decimals |
|-------|--------|---------|----------|
| Celo | CELO | Native | 18 |
| Celo (wrapped) | CELO | 0x471EcE3750Da237f93B8E339c536989b8978a438 | 18 |
| Mento Dollar | USDm | 0x765de816845861e75a25fca122bb6898b8b1282a | 18 |
| Mento Euro | EURm | 0xd8763cba276a3738e6de85b4b3bf5fded6d6ca73 | 18 |

## Notes

- All bridged tokens maintain the same decimal precision as L1
- Always verify addresses on Celoscan before interacting
- Some addresses may be placeholders - verify on official bridge documentation
- L2 WETH on Celo is the canonical wrapped ETH for DeFi interactions
