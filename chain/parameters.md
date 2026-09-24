---
description: The Robinhood Chain (chain ID 4663) settings Spatial relies on, the standards in its contracts, the Chainlink services it uses, and its timekeeping conventions.
---

# Chain parameters

Each value listed here is one that Spatial's contracts or platform rely on.

{% hint style="info" %}
For RPC endpoints, explorer URLs and Robinhood's own contract addresses, see [docs.robinhood.com/chain](https://docs.robinhood.com/chain/). Spatial's addresses are produced by the deploy script in `contracts/deployments/4663.json`, with an explanation on [Build and deploy](../architecture/build-and-deploy.md). Before sending funds, confirm any address against both sources.
{% endhint %}

## Network

| Item | Detail |
| --- | --- |
| Name | Robinhood Chain |
| Chain ID | 4663 |
| Stage | Mainnet since 1 July 2026 |
| Spatial deployment | Production |

## Execution layer and settlement

| Item | Detail |
| --- | --- |
| Execution | Arbitrum Nitro, EVM-equivalent, on ArbOS 40 with EIP-7702 support |
| Settlement layer | Ethereum |
| Gas paid in | ETH |
| Data availability | Ethereum blobs (rollup mode) |
| Block time | About 250 ms |
| Latency with preconfirmations | About 100 ms |
| Typical fee (median) | About $0.001 per transaction |
| Withdrawal via the canonical bridge | 7 days |
| Sequencer | Robinhood operates a single sequencer and screens it for sanctions |
| Forced inclusion | Through the L1 delayed inbox once 24 hours have passed |
| Block explorer | Blockscout |

## Standards in the contracts

| Standard | Usage |
| --- | --- |
| ERC-20 | Stock Tokens (18 decimals) and USDG (6 decimals) |
| ERC-8056 | Stock Tokens expose `uiMultiplier()`, which the contracts read to follow corporate actions |
| ERC-721 | Lender positions as slice tokens: the `SliceNote` contract, named "Spatial Slice Note" with symbol `SPSN` |
| EIP-712 | Typed `LendOffer` and `BorrowRequest` structs, domain `Spatial`, version `1` |
| EIP-1271 | Smart-account signatures, verified via `SignatureChecker` |
| ERC-4337 | Smart-account operations; EntryPoints v0.6 to v0.8 are deployed on the chain |
| EIP-7702 | Delegation of an EOA to smart-account code |
| ERC-20 approvals | `safeTransferFrom` pulls collateral and principal after an earlier approval; Permit2 is not used by the hub |
| Attestations | `CredentialRegistry`, an EAS-style registry built by Spatial, sitting behind the permission registry |

## Chainlink services on 4663

| Service | Purpose |
| --- | --- |
| Data Feeds for equities, 24/5 | Prices for origination and for health factors |
| Data Streams on the v11 RWA schema | Prices for auctions, with market status included in each report |
| Sequencer Uptime Feed | Triggers the liquidation grace period following an outage |
| CCIP | Collateral bridged from other chains (planned) |
| ACE and CCID | A candidate alternative adapter for eligibility |

## Time and calldata conventions on Arbitrum

| Item | How Spatial handles it |
| --- | --- |
| `block.timestamp` | The sole time source in the contracts, used for interest accrual, expiry, grace windows and the curves auctions follow |
| `block.number` | Never used as a clock, since on Arbitrum it gives the L1 block number |
| `ArbSys.arbBlockNumber()` | The precompile that returns L2 block height; nothing in the contracts needs it, so it is not called |
| Calldata | Offer and request structs pack small integer types (fields of 16, 32 and 40 bits), as the L1 data portion makes up most of the fee |
