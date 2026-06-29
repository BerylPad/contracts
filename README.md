# BerylPad Launchpad Contracts

> **B20-precompile launchpad — a fork of [clanker-devco/v4-contracts](https://github.com/clanker-devco/v4-contracts)**
> @ `b004c2edda29fa282a16d5d1441a26484f70b37f` (MIT). See [`LICENSE`](./LICENSE)
> and [Attribution & naming](#attribution--naming).

## Overview

The on-chain launchpad for **[Beryl](https://github.com/BerylPad)** — a
"DexScreener-for-B20" indexer/explorer for Base's native **B20** token standard
(Beryl hardfork). These contracts let anyone deploy a B20 token in a single
transaction with a Uniswap v4 pool, locked LP, MEV protection, optional
distribution extensions, and B20 **policy-compliance** (allow/block/frozen)
orchestration.

This is a fork of Clanker v4 whose one structural change is the **token**: instead
of deploying a bespoke `ClankerToken` ERC-20, the factory creates Base's native
**B20** via the `createB20` precompile. The entire downstream apparatus — the v4
hook, LP locker, MEV modules, and the Vault / DevBuy / Airdrop / Presale
extensions — is **unchanged** in logic, because they touch the token only through
standard ERC-20 selectors (`approve` / `transfer` / `transferFrom` / `balanceOf`).

> **Status:** **live on Base mainnet (8453)** and Base Sepolia (84532) — apparatus +
> add-on extensions deployed; B20 launch + v4 pool + locked LP + buy/sell round-trip
> all proven end-to-end.

## Contract Architecture

### Core
| Contract | Description |
|---|---|
| `BerylPad` | Token factory — orchestrates `deployToken` (B20 create → pool init → LP placement → MEV init → extensions). |
| `utils/BerylPadDeployer` | **Beryl rewire:** `createB20` + `batchMint` the full supply to the factory. The one file carrying real fork logic. |
| `BerylPadFeeLocker` | Escrow for LP fees with a per-depositor allowlist. |
| `utils/OwnerAdmins` | Owner + admin access control used across the apparatus. |

### Hooks (Uniswap v4)
| Contract | Description |
|---|---|
| `hooks/BerylPadHookV2` | Base hook — pool init, swap callbacks, LP-fee sweep, MEV coordination. |
| `hooks/BerylPadHookStaticFeeV2` | Static LP-fee strategy (apparatus default). |
| `hooks/BerylPadHookDynamicFeeV2` | Dynamic LP-fee strategy. |
| `hooks/BerylPadPoolExtensionAllowlist` | Per-pool extension allowlist. |

### LP Lockers
| Contract | Description |
|---|---|
| `lp-lockers/BerylPadLpLockerMultiple` | Locks LP, multi-recipient reward distribution (apparatus default). |
| `lp-lockers/BerylPadLpLockerFeeConversion` | Fee-conversion locker variant. |

### Extensions (optional, per launch)
| Contract | Description |
|---|---|
| `extensions/BerylPadVault` | Lock/vest a bps of supply for later release. |
| `extensions/BerylPadAirdrop` · `BerylPadAirdropV2` | Merkle-based airdrop. |
| `extensions/BerylPadUniv4EthDevBuy` · `BerylPadUniv3EthDevBuy` | Dev-buy from the pool at launch. |
| `extensions/BerylPadPresaleAllowlist` · `BerylPadPresaleEthToCreator` | Allowlist / ETH-to-creator presale. |

### MEV Modules
| Contract | Description |
|---|---|
| `mev-modules/BerylPadMevBlockDelay` | Block-delay sniper protection (apparatus default). |
| `mev-modules/BerylPadMevTimeDelay` · `BerylPadMevDescendingFees` · `BerylPadSniperAuctionV0/V2` | Alternative MEV strategies. |

### Beryl periphery (new — not vendored from Clanker)
| Contract | Description |
|---|---|
| `periphery/B20PolicyOrchestrator` | On-chain orchestrator that binds a B20's policy-compliance (allow/blocklist) and folds the fee-path infra into every compliant allowlist so a pool can't brick on the hook's fee sweep. Holds authority, not funds. |

## Deployed Contracts (Base Mainnet · 8453)

Apparatus + add-ons deployed by [`script/DeployApparatus.s.sol`](./script/DeployApparatus.s.sol)
and [`script/DeployExtensions.s.sol`](./script/DeployExtensions.s.sol).

| Contract | Address |
|---|---|
| BerylPad (factory) | [`0xF6Aa5519F2fd45aE12E3B4E0Ac4A0cBC8b37C338`](https://basescan.org/address/0xF6Aa5519F2fd45aE12E3B4E0Ac4A0cBC8b37C338) |
| BerylPadHookStaticFeeV2 | [`0x130b18BAA7F6f0d7B3730Acc605AAbe56Df028cc`](https://basescan.org/address/0x130b18BAA7F6f0d7B3730Acc605AAbe56Df028cc) |
| BerylPadLpLockerMultiple | [`0xcc248f94191691fF31f8Bee4d5a0E9816bC4eDD4`](https://basescan.org/address/0xcc248f94191691fF31f8Bee4d5a0E9816bC4eDD4) |
| BerylPadFeeLocker | [`0x4eA22306Ed7d6B5894675330E5B3C39674039FC0`](https://basescan.org/address/0x4eA22306Ed7d6B5894675330E5B3C39674039FC0) |
| BerylPadPoolExtensionAllowlist | [`0xFd7a6FF3D2c7df3eEC3bD4871f7881E4bAA3c453`](https://basescan.org/address/0xFd7a6FF3D2c7df3eEC3bD4871f7881E4bAA3c453) |
| BerylPadMevBlockDelay | [`0xeCEeFE1978d7437ba21bf72528B49c569f5AdA2b`](https://basescan.org/address/0xeCEeFE1978d7437ba21bf72528B49c569f5AdA2b) |
| B20PolicyOrchestrator | [`0xc52cF2E98644b0755F1769952A90856c387FA6B2`](https://basescan.org/address/0xc52cF2E98644b0755F1769952A90856c387FA6B2) |

### Add-on extensions + MEV modules (Base Mainnet)

| Contract | Address |
|---|---|
| BerylPadVault | [`0x4E62786686d6d2A4D4Af630C2BEE5c2958fD8b0d`](https://basescan.org/address/0x4E62786686d6d2A4D4Af630C2BEE5c2958fD8b0d) |
| BerylPadAirdrop | [`0xe4426Ce9e0E6D62aEd19f1aA1aeE2E287FD853CC`](https://basescan.org/address/0xe4426Ce9e0E6D62aEd19f1aA1aeE2E287FD853CC) |
| BerylPadAirdropV2 | [`0xEDD6D045bAdfdCa2fbE22EaCcf83206E7D628b95`](https://basescan.org/address/0xEDD6D045bAdfdCa2fbE22EaCcf83206E7D628b95) |
| BerylPadUniv4EthDevBuy | [`0xb0c866CCa8DAF3b9555700D7474bd300F7b97c37`](https://basescan.org/address/0xb0c866CCa8DAF3b9555700D7474bd300F7b97c37) |
| BerylPadUniv3EthDevBuy | [`0xe58973E5e6BF5d27a94c8d3df190541ECb50Bb23`](https://basescan.org/address/0xe58973E5e6BF5d27a94c8d3df190541ECb50Bb23) |
| BerylPadMevTimeDelay | [`0xCD305CB3005b9ef6068d2f5a590d1A6835054605`](https://basescan.org/address/0xCD305CB3005b9ef6068d2f5a590d1A6835054605) |
| BerylPadMevDescendingFees | [`0xE75Ed2E27De786d411b564f7232EC8Ec81C9fdBB`](https://basescan.org/address/0xE75Ed2E27De786d411b564f7232EC8Ec81C9fdBB) |
| BerylPadSniperAuctionV2 | [`0x6Bde6e5CbfcAe241763F3E4f6aeC0fb4CAa70f8c`](https://basescan.org/address/0x6Bde6e5CbfcAe241763F3E4f6aeC0fb4CAa70f8c) |

### External dependencies (Base Mainnet)
| Contract | Address |
|---|---|
| Uniswap v4 PoolManager | `0x498581fF718922c3f8e6A244956aF099B2652b2b` |
| Uniswap v4 PositionManager | `0x7C5f5A4bBd8fD63184577525326123B519429bDc` |
| Permit2 | `0x000000000022D473030F116dDEE9F6B43aC78BA3` |
| WETH | `0x4200000000000000000000000000000000000006` |
| B20 PolicyRegistry (precompile) | `0x8453000000000000000000000000000000000002` |

## Deployed Contracts (Base Sepolia · 84532)

Apparatus + add-ons deployed by [`script/DeployApparatus.s.sol`](./script/DeployApparatus.s.sol)
and [`script/DeployExtensions.s.sol`](./script/DeployExtensions.s.sol).

| Contract | Address |
|---|---|
| BerylPad (factory) | [`0x60eA8997B765dE0C457d88c37FB31D465131169c`](https://sepolia.basescan.org/address/0x60eA8997B765dE0C457d88c37FB31D465131169c) |
| BerylPadHookStaticFeeV2 | [`0x50e0218b9a385C2D10eE5e5c0578F2A33Ac5e8Cc`](https://sepolia.basescan.org/address/0x50e0218b9a385C2D10eE5e5c0578F2A33Ac5e8Cc) |
| BerylPadLpLockerMultiple | [`0xEC7Cf527cbb5150E35d39C00a407e0EAEc29Ce3C`](https://sepolia.basescan.org/address/0xEC7Cf527cbb5150E35d39C00a407e0EAEc29Ce3C) |
| BerylPadFeeLocker | [`0x07F42908202E06BBECd265e00B117E8C70C47E66`](https://sepolia.basescan.org/address/0x07F42908202E06BBECd265e00B117E8C70C47E66) |
| BerylPadPoolExtensionAllowlist | [`0xB398096240Ec29E8c4432a71abD66c8A839eb166`](https://sepolia.basescan.org/address/0xB398096240Ec29E8c4432a71abD66c8A839eb166) |
| BerylPadMevBlockDelay | [`0x70BA5e996160DB5F1a9fC6545BebC3E9BF666960`](https://sepolia.basescan.org/address/0x70BA5e996160DB5F1a9fC6545BebC3E9BF666960) |
| B20PolicyOrchestrator | [`0x5EfACBdeE2aC17fC601962d5D2cadDbDE8103B7f`](https://sepolia.basescan.org/address/0x5EfACBdeE2aC17fC601962d5D2cadDbDE8103B7f) |

### Add-on extensions + MEV modules (Base Sepolia)

| Contract | Address |
|---|---|
| BerylPadVault | [`0xe7A7c167ADe76c2aBeb02A737D3abD94eAe32e6b`](https://sepolia.basescan.org/address/0xe7A7c167ADe76c2aBeb02A737D3abD94eAe32e6b) |
| BerylPadAirdrop | [`0x27E51280FE8fC0157B3061Ffbe9831461faBB8b5`](https://sepolia.basescan.org/address/0x27E51280FE8fC0157B3061Ffbe9831461faBB8b5) |
| BerylPadAirdropV2 | [`0xE3a90260E8cEB63338AbaeA8306cD19d69fCD1b2`](https://sepolia.basescan.org/address/0xE3a90260E8cEB63338AbaeA8306cD19d69fCD1b2) |
| BerylPadUniv4EthDevBuy | [`0x5fe1c58c24522eA319c53247a04d1fAb7BA9E970`](https://sepolia.basescan.org/address/0x5fe1c58c24522eA319c53247a04d1fAb7BA9E970) |
| BerylPadUniv3EthDevBuy | [`0x5EAcEd7432C3BaD614dB87A361E7276e500B7c3E`](https://sepolia.basescan.org/address/0x5EAcEd7432C3BaD614dB87A361E7276e500B7c3E) |
| BerylPadMevTimeDelay | [`0x8DfF2A9f2E5EaeC8038Ad4C0BF9D801AE8C2ED2A`](https://sepolia.basescan.org/address/0x8DfF2A9f2E5EaeC8038Ad4C0BF9D801AE8C2ED2A) |
| BerylPadMevDescendingFees | [`0xa22f9137982E0aa99072dE4c02Eb59D7F9B530e9`](https://sepolia.basescan.org/address/0xa22f9137982E0aa99072dE4c02Eb59D7F9B530e9) |
| BerylPadSniperAuctionV2 | [`0x49E54546808099E6feefD0AD46E2eb1a1c0F8A4d`](https://sepolia.basescan.org/address/0x49E54546808099E6feefD0AD46E2eb1a1c0F8A4d) |

### External dependencies (Base Sepolia)
| Contract | Address |
|---|---|
| Uniswap v4 PoolManager | `0x05E73354cFDd6745C338b50BcFDfA3Aa6fA03408` |
| Uniswap v4 PositionManager | `0x4B2C77d209D3405F41a037Ec6c77F7F5b8e2ca80` |
| Universal Router (v4) | `0x492E6456D9528771018DeB9E87ef7750EF184104` |
| Permit2 | `0x000000000022D473030F116dDEE9F6B43aC78BA3` |
| WETH | `0x4200000000000000000000000000000000000006` |

## What this fork changes vs upstream

| File | Change |
|---|---|
| `src/utils/BerylPadDeployer.sol` | **Rewired:** `new ClankerToken{salt}(…)` → `StdPrecompiles.B20_FACTORY.createB20(ASSET, salt, encodeAssetCreateParams(…), initCalls)` where `initCalls` `batchMint`s the full supply to the factory (delegatecall context). Originating-chain mint gate preserved. |
| `src/ClankerToken.sol` | **Deleted** — the upstream Clanker token; B20 is a precompile, not deployable EVM bytecode. |
| `src/periphery/B20PolicyOrchestrator.sol` | **New** (Beryl) — B20 policy-compliance orchestration. |
| `src/hooks/BerylPadHook.sol`, `BerylPadHookV2.sol` | Removed the **dead** `import {ClankerToken}` (never referenced; blocked the B20-only build). |
| `foundry.toml` | `base = true` (registers the B20 / Policy / Activation precompiles in `base-forge`). |
| `remappings.txt` | Added `base-std/`; removed `@contracts-bedrock/` (Optimism — only the removed `ClankerToken` used it for `IERC7802` / Superchain crosschain). |

**Dropped capabilities** (lived on the upstream `ClankerToken`, no B20 analog):
on-chain image/metadata/context (Beryl indexes these off-chain),
`verify()`/`isVerified()`, ERC20Votes, ERC20Permit, ERC20Burnable, and the
IERC7802 Superchain cross-chain mint/burn. The B20 **gains** native policy-state
(allow/block/frozen, pause, supply-cap, rebase, memo).

## Build & test

These contracts require **Base's Foundry build (`base-forge`)** — stock Foundry
cannot simulate the B20 / Policy / Activation precompiles. The ~175 MB dependency
tree is not committed; `setup.sh` reconstructs it at the pinned upstream versions.

```bash
./setup.sh                 # reconstruct lib/ at pinned versions + base-std
BASEFORGE="$HOME/.foundry/versions/base-v1.1.0:$HOME/.foundry/bin"
PATH="$BASEFORGE:$PATH" base-forge build
PATH="$BASEFORGE:$PATH" base-forge test          # fork tests need --fork-url <base-sepolia>
```

The full Uniswap v4 + apparatus tree compiles clean against the B20 precompile
(only upstream forge-lint typecast warnings remain). Compiler: Solidity 0.8.28,
viaIR, optimizer 20,000 runs, EVM target Cancun.

## Pinned dependency versions (see `setup.sh`)

`forge-std` `77041d2` · `openzeppelin-contracts` `a7d38c7` · `permit2` `cc56ad0` ·
`universal-router` `3663f6d` · `v4-core` `5f00c84` · `v4-periphery` `9628c36` ·
`base-std` (latest). The Optimism monorepo is intentionally **not** fetched.

## Contributing

See [`CONTRIBUTING.md`](./CONTRIBUTING.md) for setup, build/test/format
requirements, and the security-critical paths. Report vulnerabilities privately
per [`SECURITY.md`](./SECURITY.md) — never open a public issue.

## Attribution & naming

Forked from [Clanker v4](https://github.com/clanker-devco/v4-contracts) by Clanker
Devco, licensed under MIT (per-file SPDX headers preserved). See [`LICENSE`](./LICENSE)
for the dual copyright notice.

**On naming:** the vendored `Clanker*` contracts are renamed to `BerylPad*` for
brand consistency. The reference ABIs under [`base_mainnet_abis/`](./base_mainnet_abis)
keep their original `Clanker*` names — they document Clanker's **live mainnet**
contracts and are not ours. The genuinely Beryl-specific code is
`B20PolicyOrchestrator` and the `BerylPadDeployer` B20 rewire; everything else is
vendored Clanker v4 logic under a renamed identifier.

## License

MIT — see [`LICENSE`](./LICENSE).
