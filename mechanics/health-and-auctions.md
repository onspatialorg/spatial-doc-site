---
description: How Spatial measures loan health, how LiquidationAuction sells collateral at a falling price, and what changes when markets are shut or the sequencer goes down.
---

# Loan health and the liquidation auction

## The health factor

Call `healthFactor(loanId)` on the hub and it computes:

```
HF = collateralValue × liquidationLtvBps / 10_000 / debt
```

* `collateralValue` is the escrowed token amount multiplied by the `OracleGuard` price, expressed in USDG.
* `liquidationLtvBps` is the liquidation LTV for the tier (a different figure from the max LTV applied when the loan was opened), reduced by the closed-market haircut whenever the quote is stale or was taken while the session was closed.
* `debt` is principal plus interest due over all slices, accurate to the second.

There are two lines to watch:

| Health factor | Name | Consequence |
| --- | --- | --- |
| under 1.10 | Warning | The borrower is alerted by keepers and the platform. Adding collateral or repaying part of the loan will lift it. |
| under 1.00 | Liquidation | Any address is free to call `startAuction(loanId)`. |

Health is not the only trigger. A loan may be auctioned regardless of its health factor if it is Defaulted, which happens when its rollover auction failed, or if it is still Active after maturity and the 24-hour grace period have both passed.

## Starting an auction

No permission is needed to call `startAuction`, and whoever calls it is recorded as the keeper. The call works through these steps in turn:

1. It fetches a new price through `OracleGuard`, which stores a move-cap checkpoint.
2. It confirms the loan is eligible for auction.
3. It switches the loan to Liquidating, cancelling any rollover that is running.
4. During a closed session, it splits off slices whose lenders opted out.
5. It settles any in-kind slices.
6. Where no collateral remains for sale, it finalizes with zero proceeds. In every other case it records the schedule of prices and emits `AuctionStarted`.

The call reverts if the token is paused, if the sequencer is still in its grace period, if the feed has gone stale while the market is trading, or if liquidations are paused.

## How the price falls

| `RiskConfig` field | Value | Meaning |
| --- | --- | --- |
| `startPremiumBps` = 300 | quote × 1.03 | Starting price |
| `floorBpsRegular` = 7000 | quote × 0.70 | Lowest price in regular or extended hours |
| `floorBpsClosed` = 8500 | quote × 0.85 | Lowest price while the market is closed |
| `duration` | 45 minutes, falling linearly | Length of the window |
| `penaltyBps` = 300 | debt with the 3% penalty added | Amount the auction aims to raise |

`currentPrice(loanId)` declines over the window until it reaches the floor, then stays there until a purchase or a restart.

### Purchases

Calling `buy(loanId, collateralAmount, streamReport)` requires the LIQUIDATOR role. The buyer pays USDG to the hub and receives collateral from escrow at the prevailing price, all at once or across several purchases. Each purchase is trimmed in two ways: it cannot exceed the collateral left in escrow, and it cannot cost more than the remainder of the target. As a result an auction never raises more than the debt plus the penalty. For tokens that have a Data Streams adapter, the caller must include a signed report, which the guard verifies before the sale goes through. Even so, the sale price is always taken from the schedule, and Chainlink Data Feeds set that schedule in every case. The auction ends, and the hub finalizes it, once proceeds meet the target or no collateral is left.

### Restarts

If the window expires before the target is met, anyone can reprice the auction with `restart(loanId)`. A fresh quote is used to rebuild the starting price, floor, target and window. Proceeds already raised are kept, and the keeper who opened the auction still receives the keeper share. The liquidations pause blocks `buy` and `restart` in addition to new auctions. Once the pause is lifted, a restart produces a new schedule, so no one can buy at prices fixed before the pause began.

### Why sell by auction instead of swapping

Pooled lending markets tend to liquidate through an AMM. Single-name Stock Token pools have little depth, though, and a forced sale through one would shift the price much further than the penalty covers. A falling, quoted price invites anyone with a view on fair value to bid: the lenders on the loan, Robinhood's authorised participants, liquidity providers on Morpho and Uniswap, and DEX arbitrageurs.

## Penalty and payout order

The penalty equals 3% of the debt (`penaltyBps` = 300). With `keeperShareBps` = 3334, one third of it, a single point, is paid to the keeper. The remaining two points are split by `lenderPenaltyShareBps` = 5000: half to the lenders, half to the protocol.

`finalizeLiquidation` pays out proceeds in the sequence below, and no tranche can take more than what remains:

1. one point to the keeper,
2. lender claims for principal and interest, divided by each slice's share of the debt,
3. one penalty point to the lenders,
4. one penalty point to the protocol,
5. any leftover to the borrower.

The 10% interest share applies only to interest the auction actually recovers, and it goes to the FeeVault together with the protocol's penalty point. Collateral that was not sold goes back to the borrower. The loan always ends as Settled, even if lenders did not recover everything. Any shortfall falls on the slices in proportion to their size, and once tranching from the roadmap ships, on junior slices before others. Other loans are unaffected. The record is closed by `LiquidationFinalized`.

## Taking collateral instead of cash

A lender whose offer had `selfLiquidate` set receives collateral rather than a share of the sale, priced at the checkpointed quote. `settleInKind` converts what the slice is owed, principal and interest alike, into tokens at that price. It can take no more than the slice's share of the escrow by principal weight, which protects the collateral backing every other slice. No interest share is taken. Whatever exceeds the cap is written off as the slice closes, and it leaves the auction before any tokens are sold. This option is meant for lenders who would rather hold the shares.

## When the market is closed

Stock markets close every night and at weekends, and the Monday open may sit far away from where Friday ended. Three rules kick in when the session is reported as closed, or when the feed's `updatedAt` is older than the staleness limit for that session:

* **Haircut.** A tier-specific cut of 10 to 15 points is subtracted from both the origination ceiling and the liquidation LTV used for the health factor.
* **Raised floor.** Auctions may still open, but the floor rises from 0.70 to 0.85 × quote, which keeps a weekend sale from turning predatory.
* **Opting out.** A lender who sets `noClosedMarketLiquidation` keeps their slice out of any auction started while the market is closed. The opted-out slices, carrying the collateral that matches their weight by principal, are shifted into a freshly created Active loan, owned by that borrower and keeping the original dates, and only the slices that remain are sold. If all slices opted out, there is nothing to sell, so `startAuction` fails with `NoClosedMarketLiquidation` until trading resumes.

## Following a sequencer outage

`OracleGuard` reads Chainlink's L2 Sequencer Uptime Feed. Once the sequencer is back up, there is a one-hour period (`sequencerGrace`) during which no auction and no origination can start. Borrowers can use it to top up collateral, either directly or through transactions they submitted via the L1 delayed inbox while the outage lasted. The [sequencer outage page](../risk/sequencer.md) covers this in full.

## Price checks run before anything else

None of the steps above go ahead on a price that `OracleGuard` refuses. It rejects:

* an answer of zero or below,
* an `updatedAt` that breaches the staleness limit of the current session,
* a move of more than 25% from one checkpoint to the next, which stops the market until someone reviews it by hand,
* any token whose issuer contract returns true for `oraclePaused()`, which signals a corporate action under way.

The [pricing page](../risk/pricing.md) explains these checks further.

## Example

Take a Tier A loan, with a 70% liquidation LTV, holding 250 NVDA as collateral against 20,000 USDG of debt including interest.

| HF | NVDA price | Status | Collateral value |
| --- | --- | --- | --- |
| 1.54 | 176.40 | healthy | 44,100 |
| 1.09 | 125.00 | warning | 31,250 |
| 0.9975 | 114.00 | auctionable | 28,500 |

A keeper starts the auction at 117.42 (that is, 114 × 1.03), with 20,600 as the target and 79.80 as the floor. When the price reaches 113.00, a liquidator bids for the full 250 tokens. The bid is cut back to the target, so the liquidator pays 20,600 USDG and receives 182.30 NVDA. With the auction closed, the keeper receives roughly 200 USDG, the slices get 20,000, the lenders take about 200 more as a penalty point, and about 200 lands in the FeeVault. The borrower gets back the 67.70 NVDA left unsold.

Suppose instead the loan had been valued on a Saturday. A 10-point haircut would bring the liquidation LTV down to 60%. To keep the health factor above 1.0 the loan would then need 33,333 in collateral value, it would become auctionable at an NVDA price of 133.33, and the floor would sit at 113.33.
