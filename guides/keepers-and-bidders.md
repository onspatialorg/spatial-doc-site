---
description: The reference keeper's routine, how to start a collateral auction and bid in it, and the boundaries the contracts set on each role.
---

# Running keepers and bidding in auctions

No operator holds special rights over Spatial's auctions. Independent parties running open-source code start auctions, buy collateral, file delegated rollover acceptances and send alerts to borrowers, and the contracts pay every reward on-chain.

## Roles and rewards

| Role | Job | Reward |
| --- | --- | --- |
| Keeper | Calls `startAuction` for eligible loans, delivers alerts, and files rollover acceptances on behalf of lenders who delegated them | Roughly 1 point of the penalty (which is 3 percent), earned by triggering |
| Liquidator | Purchases collateral in the Dutch auction | The difference between fair value and the price paid |
| Arbitrageur | Acquires collateral in the auction and hedges or sells it on another venue | That same difference |

Triggering is open to all. Holding collateral at the end, whether purchased or taken in kind, requires the `LIQUIDATOR` attestation.

## The reference keeper

Spatial publishes a reference keeper in TypeScript. It needs a single private key and keeps no other state. It reads from the public indexer and submits its own transactions, repeating these steps:

1. Subscribe to the loan stream the indexer publishes.
2. For each active loan, compute the health factor using `OracleGuard.quote()` for the session in progress.
3. Below 1.10, send the borrower's registered alerts.
4. Below 1.00, or when the loan has reached Defaulted, submit `startAuction(loanId)`, but only if the quote shows `sequencerGrace` as false.
5. For lenders who delegated rollover acceptance, track open rollover auctions and submit the acceptance they signed in advance when the rising rate reaches their limit.

You can run any number of instances. The first `startAuction` to be included is credited as the keeper and receives the share.

## How to bid

1. Listen for `AuctionStarted`.
2. Read `currentPrice(loanId)`. Its starting level is the oracle price times 1.03, and it declines linearly to the floor over about 45 minutes. During the regular session that floor is 0.70 times oracle. If the target has not been met by the end of the window, the price stays at the floor until a buyer arrives or the schedule restarts from a new quote.
3. Fetch a Chainlink Data Streams report for the token on sale.
4. Submit `buy(loanId, collateralAmount, streamReport)`. The contract verifies the report and reverts if the stream and the feed differ by more than the allowed tolerance. Partial purchases are fine. The auction ends when proceeds hit the target or no collateral remains in escrow.

In the closed session, the floor cannot go under 0.85 times oracle, and any slices carrying `noClosedMarketLiquidation` are split off into a separate loan before the auction starts.

## Transaction ordering

Robinhood Chain's sequencer orders transactions by arrival. If Timeboost is enabled later, buying the express lane could determine which `startAuction` or `buy` lands first. Should that happen, the auction design will be reviewed and any change made public. See [When the sequencer stops](../risk/sequencer.md) for more.

## What a keeper cannot do

* Liquidate a loan in good health. `startAuction` reverts if the on-chain conditions are not satisfied.
* Set the price. The curve and the floor are parameters held in `RiskConfig`.
* Act during the grace period that follows a sequencer outage.
* Take collateral without holding the eligibility attestation.
* Start an auction or buy in one while the guardian has liquidations paused.
