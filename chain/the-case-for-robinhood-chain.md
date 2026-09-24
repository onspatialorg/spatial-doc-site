---
description: The requirements a lending protocol for tokenised equities places on its chain, how Robinhood Chain meets each one, and the three gaps Spatial fills itself.
---

# Why Robinhood Chain

A chain suits securities-backed lending only if it clears three bars. The collateral must be a standard ERC-20 that holders keep in their own wallets. A regulated entity must stand behind it. Chainlink must price it. Robinhood Chain clears every one, and adds an advantage competitors cannot copy: the broker that issues the collateral is also the one that distributes it to its holders. The evidence follows below.

## Point by point

The sources are Robinhood's own published documentation and Arbitrum Foundation material.

### Collateral and prices

| Area | On Robinhood Chain | Value to Spatial |
| --- | --- | --- |
| Native RWAs | **Stock Tokens**, which are 18-decimal ERC-20s from Robinhood Assets (Jersey) Ltd, backed 1:1 by shares in US custody and sold across 120+ countries | Collateral issued directly by a listed broker |
| Corporate actions | ERC-8056 `uiMultiplier()`: dividends are reinvested and splits become a shares-per-token ratio | The collateral compounds without help, and dividends never leak out of escrow |
| Oracles | Chainlink Data Feeds running 24/5; Data Streams on the v11 RWA schema with its `marketStatus` field; CCIP; the Sequencer Uptime Feed; the multiplier is included in every price | Production-grade pricing from day one |
| Stablecoins | USDG (Paxos, regulated under MiCA), native, and what Robinhood Earn lends out through Morpho; USDe live; USDC via CCTP *unconfirmed* | The currency loans are made in |
| Token-level compliance | **Nothing at token level**: no allowlist or transfer hook on Stock Tokens; Robinhood enforces eligibility in its interface and at issuance and redemption, which require KYC | Spatial runs its own eligibility checks where users enter the protocol |

### Infrastructure

| Area | On Robinhood Chain | Value to Spatial |
| --- | --- | --- |
| Stack | A dedicated Arbitrum Nitro (Orbit) chain in rollup mode; settles on Ethereum with data in blobs; canonical withdrawals take 7 days | Security from Ethereum and Arbitrum tooling with no changes needed |
| Status | Chain ID 4663, on mainnet since 1 July 2026; about $540M TVL in mid-August 2026, though reported figures range from $536M to $1.4B | A live network with real users and real assets |
| Deploying | Permissionless, EVM-equivalent; Solidity or Vyper using Foundry or Hardhat, plus Stylus for Rust and WASM | No permission needed to ship; signature verification is in Solidity now, and Stylus is an option for a cheaper batch verifier |
| Speed and cost | Blocks roughly every 250 ms, 100 ms with preconfirmations; a typical transaction costs about $0.001 (median); ETH pays for gas | Quick originations and liquidations, and filling twenty offers at once remains inexpensive |
| Sequencer | Robinhood runs one sequencer, ordering first come first served and screening for sanctions; if it censors, the L1 delayed inbox lets transactions in once 24 hours pass | Spatial is designed to withstand a full day of downtime, and can read sequencer status from a Chainlink uptime feed |
| Smart accounts | EIP-7702 plus ERC-4337 EntryPoints from v0.6 up to v0.8 (ArbOS 40), served by Alchemy Gas Manager and ZeroDev | Gas-free onboarding, approve and borrow in a single batch, and offer signatures via EIP-1271 |
| Existing DeFi | Morpho Blue; Uniswap in versions 2, 3 and 4; 1inch, Lighter and Arcus | Somewhere for unmatched capital to wait, plus liquidity that auctions can clear into |
| Economics | 10% of net chain revenue goes from Robinhood to the Arbitrum DAO, and Robinhood also committed $1M to sponsor Arbitrum Open House 2026 | Grants and audit subsidies are open to builders |

## The alternatives

* **Solana.** Stocks tokenised on Solana (Kamino, Jupiter Lend) live in pooled markets with strict caps, and they come from third-party issuers rather than from the broker whose customers own the shares.
* **Base.** Coinbase launched tokenised stocks on Base on 24 August 2026. They are new, and no lending stack has formed around them so far.
* **Ethereum mainnet.** It has the deepest DeFi anywhere but no broker-issued equities of its own, and its gas costs make a twenty-lender syndicate unviable.

Robinhood Chain is the only place that combines the assets, the feeds, the loan currency, cheap fees and account abstraction, all backed by the distribution of the broker that issues them.

## Gaps Spatial fills

An honest case names what is missing. Three things are, and Spatial handles each.

1. The chain does not restrict who can hold a Stock Token. For Spatial, the [permission registry](../architecture/eligibility.md) takes on that job.
2. The chain offers users no protection when the sequencer halts. Spatial offers the delayed-inbox route plus a grace period before liquidation, described in [Sequencer outages](../risk/sequencer.md).
3. The chain does not handle the price gap that builds up while markets are shut. Spatial uses haircuts that depend on the trading session; see [pricing and sessions](../risk/pricing.md).
