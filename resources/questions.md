---
description: Straight answers to the things people most often want to know about Spatial.
---

# Frequently asked questions

## Borrowing and collateral

**What can I pledge?**
For now, Tier A and Tier B Robinhood Stock Tokens, with the tier set by liquidity. Next on the roadmap come long-tail Stock Tokens, followed by fund shares and tokenized treasuries. The [tiers page](../collateral/tiers-and-ltv.md) has the specifics.

**What do I borrow?**
USDG, the Global Dollar that Paxos issues.

**How is the rate set?**
No single party sets it. A borrow request states the maximum APR the borrower will pay, and each lend offer states the minimum its lender will accept. When the two meet, the hub originates the loan, and that rate is fixed until the term ends.

**Can I pay off a loan early?**
Yes, after 3 days. Interest is charged for at least those first 3 days, which is the minimum interest period.

**My term is ending. Now what?**
You have three paths. Pay everything back. Send the loan into a rollover auction, in which lenders compete to extend it. Or let it lapse, in which case it goes into default and a liquidation auction sells the collateral. [Maturity, grace and rollover](../mechanics/maturity-and-rollover.md) explains each.

**Is it different when the stock market is closed?**
It is. A closed-market haircut kicks in, which lowers your effective liquidation LTV, and any auction's floor is bounded. Check your closed-market health factor before Friday's close.

## For lenders
**Do I pay gas to post an offer?**
No. An offer is a signed message rather than a transaction. Gas is paid when the loan originates, by the borrower or on their behalf via account abstraction.

**What do I own after a fill?**
A slice token, which is an ERC-721 standing for your part of the loan. You can transfer it to any other eligible address.

**What does Spatial take?**
10% of the interest you earn, deducted from the payout. Your principal is never charged.

**Is my principal at risk?**
On your own slices, yes, if a liquidation auction realises less than you are owed. Losses on loans you did not fund never touch you, because every slice is isolated from the others.

## When things go wrong

**What if the price feed misbehaves?**
`OracleGuard` deals with it. A price of zero reverts. A stale reading or a paused market gets flagged, and the hub refuses to originate or liquidate against it. If the price jumps by more than the configured cap, the market is paused. If a Data Streams adapter is available, the stream price is compared with the Data Feed. The feed always tracks the exact token held in escrow, never a wrapper around it. There is more on the [pricing page](../risk/pricing.md).

**What if the sequencer goes down?**
After it recovers, liquidations stay paused for a grace period. During the outage itself, the delayed inbox on L1 keeps every function reachable. [When the sequencer stops](../risk/sequencer.md) has more.

**Can the issuer freeze Stock Tokens in escrow?**
Before each Stock Token's market opened, its deployed bytecode was checked for any role able to pause or freeze it, put holders on a blacklist or push through a forced transfer. What was found is published next to the market on Telemetry, the public risk page. If one turned up, the tier's LTV and exposure cap were set to allow for it. The [issuer page](../risk/issuer.md) covers this.

## Who can use it, and is it live
**Is Spatial live?**
Yes. It runs in production on chain ID 4663, Robinhood Chain. You can open the platform at [onspatial.org/platform](https://onspatial.org/platform).

**Is Spatial a lending pool?**
No. It pairs one lender at a time with one borrower at a time on loans with a fixed rate and a fixed term, and each slice is independent. Lenders can keep unmatched capital in a Morpho vault, but that vault belongs to Morpho and is not a Spatial pool.

**Who can use it?**
Professional lenders, plus borrowers who are verified businesses or individuals who meet the professional or high-net-worth test, in each case within an eligible jurisdiction. Retail users are added jurisdiction by jurisdiction as regulation allows.

**Does being a US person rule me out?**
No. What matters is verification, not where you live. Get through the suitability check and identity verification, and your wallet receives an attestation.

## Building and running infrastructure

**Can I operate my own indexer or relayer?**
Yes. Both are open source and permissionless. The order book can be rebuilt from signed messages, and loan state from chain events.

**Can anyone run a keeper?**
Anyone can. [Keepers, liquidators and bidders](../guides/keepers-and-bidders.md) is the place to begin.

**Where are the contract addresses?**
In the repository at `contracts/deployments/4663.json`. The Explorer shows the same addresses, and so does the platform alongside its protocol parameters. [Build and deploy](../architecture/build-and-deploy.md) has more.
