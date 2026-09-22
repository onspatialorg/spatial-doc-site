---
description: A single sequencer orders transactions on Robinhood Chain. This page covers how it might fail and four safeguards that keep a Spatial position repayable, top-up-able and cancellable through each of them.
---

# If the sequencer goes down

Borrowers and lenders need to keep control of a loan even if the chain under it falls silent. Robinhood Chain is an Arbitrum Nitro rollup, and Robinhood operates its one sequencer, which is typical for an L2 at this stage. That setup comes with well-understood ways to fail, and Spatial is designed to keep working through all of them.

## The four defences

### Uptime feed and a one-hour grace window

`OracleGuard` reads the L2 Sequencer Uptime Feed from Chainlink, configured through `setSequencerFeed`. A reading of 0 means the sequencer is up and 1 means it is down, while `startedAt` records the moment the present state started. Once the sequencer comes back, every quote is marked with the `sequencerGrace` flag for `sequencerGrace` seconds, which the deployed loan parameters set to one hour. During that window auctions cannot open and loans cannot originate: a price that no one was able to trade on through the outage should not yet be used to liquidate anyone. The hour is for borrowers to top up collateral or pay back. If the sequencer is still offline, that is treated as grace too, and so is an uptime round lacking a usable start time, meaning a faulty feed leans towards stopping liquidations rather than allowing them.

### Forcing a transaction in via the delayed inbox

Arbitrum maintains a delayed inbox on Ethereum, and every function that changes state is reachable through it. If the sequencer has not included such a transaction after 24 hours, any account can force its inclusion. As a result:

* a censoring sequencer has no way to stop a borrower from calling `repay`,
* whatever the sequencer does, a lender can still invalidate an offer nonce with `cancel` or `cancelWord`,
* a borrower expecting an outage can submit `addCollateral` through L1 ahead of time.

Taken together with the grace window, this gives users a route they can genuinely rely on, not an afterthought.

### Timing taken from timestamps, not block numbers

On chains built on the Arbitrum stack, `block.number` returns a figure linked to L1 instead of the L2's own block count. For that reason accrual, expiry and auction timing all run on `block.timestamp`, and `block.number` is not read anywhere. If a true L2 block height were ever needed, it would come from `ArbSys.arbBlockNumber()`, though no current contract uses it.

### Compact calldata

What a transaction pays for its L1 data component depends on how far its calldata compresses. Offer structs are packed tightly, and compact 64-byte signatures are accepted wherever the signer supports them. As a result an `originate` call that bundles twenty offers remains cheap.

## Failure modes

| What fails | What users experience |
| --- | --- |
| Sequencer outage | Nothing gets onto the L2 until the sequencer is back, while stocks keep trading in the meantime. |
| Sequencer censorship | Specific transactions, such as a repayment, are never included. |
| Slow finality on L1 | Withdrawing to Ethereum takes the usual 7 days. Loans on the L2 are not affected. |
| Changes to chain configuration | Switching DA mode, changing the validator set, or turning on Timeboost or BoLD could change the ordering and finality assumptions the protocol relies on. |

## Tracking chain configuration

Whether Timeboost (ordering through an express lane) and BoLD (validation open to anyone) are active on Robinhood Chain affects two things: which liquidator gets to an auction first, and how quickly a force-included transaction counts as final. Spatial keeps track of both. Should Timeboost be switched on, the auction mechanics will be reviewed to make sure the express lane cannot routinely be used to get ahead of keepers, and any change that results is recorded in the governance log. The chain's live values are listed on [Network parameters](../chain/parameters.md).
