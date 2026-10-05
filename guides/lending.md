---
description: How to post standing and targeted offers from the Lend screen, how the three flags on an offer work, and what each slice returns to you.
---

# Lending at your own price

Spatial has no pool. You choose the collateral, the highest LTV you accept, the rate and the term, and each position you hold is independent of the rest. When a loan you did not price goes bad, the loss belongs to whoever priced it.

## Who can lend

Professional clients receive the `LENDER_PROFESSIONAL` attestation. Think of market makers, family offices, credit funds working in crypto, and firms of that kind. Retail lending is being added jurisdiction by jurisdiction wherever regulation permits, and [Jurisdictions](../compliance/jurisdictions.md) has the current list.

Beyond that you need a wallet on Robinhood Chain and some USDG held in either of two ways. It can be approved to `CreditHub` (Permit2 approvals are accepted), or it can sit in the whitelisted Morpho vault, in which case your offers carry `parkIdle`.

## Using the Lend screen

At the top are three figures. Lend capacity sits beside Open requests and Open demand. Underneath, "Open borrow requests" acts as the order book, and you can filter it by collateral or by term. A row gives you the borrower's address, the collateral and its tier, the principal, the LTV, the Max APR, the term and how much of the launch window is left.

### Offers aimed at one request
Click "Make offer" on any row and the form switches to a "Targeted offer". The collateral, the term and the request are fixed, and the APR starts at the borrower's maximum. Once you sign, the offer exists: your signature is all it takes, and you pay no gas.

### Offers that stand for a tier
When you open the form it is already set up for "Post a standing offer". You can aim it at a single token ("This token only") or at every token in a tier ("Whole tier"). You set the APR, a cap on principal, the smallest fill you will take, a maximum LTV that cannot exceed the tier's limit, the term, and how many days until it expires. It behaves like a resting limit order, and requests keep drawing from it until the capacity runs out or the expiry arrives. Most of the book's depth comes from standing offers, and they are how a professional lender usually operates.

### The three flags

At the bottom of the form are three switches, and each one turns on a bit in the `flags` field of your offer.

| Switch | Flag | Effect |
| --- | --- | --- |
| Take collateral in kind | `selfLiquidate` | If the loan is liquidated, you are paid in collateral, priced off the oracle, with a cap set at the proportion of the escrow your slice represents, rather than waiting on auction proceeds. |
| No closed-market liquidation | `noClosedMarketLiquidation` | Your slice is excluded from any auction started while equity markets are shut. The gap risk stays with you. |
| Park idle capital | `parkIdle` | USDG that has not been matched remains in the Morpho vault, and at origination the hub pulls out only the amount filled. |

Press "Sign and post offer" and the offer appears in the "Your offers" table, which shows the market or tier, the capacity left, APR, term, expiry and status, plus a "targeted" badge on targeted offers.

## Your slices

Every fill becomes a slice, and you receive it as a slice token. On the Positions screen, "Slices you hold" gives each one's loan and collateral, the principal and APR that are yours, the interest accrued so far (it accrues per second, at your APR), plus the health factor, status and maturity of the loan. You can also see whether the collateral's market is open, how old the oracle price is, whether a rollover auction is running, which other lenders share the loan, and what other slices that borrower has.

## How you get paid

| What happens | What you receive |
| --- | --- |
| The loan is repaid in full | Your principal plus your slice's interest, minus the 10 percent protocol share, sent within the borrower's repayment transaction. |
| The loan is partly repaid | Your principal shrinks by your slice's portion of the amount repaid. |
| A rollover clears | All that is due to you, principal and accrued interest, as soon as the new syndicate has funded. If you accept the new rate, you stay in the loan instead. |
| The loan is liquidated | Your pro rata portion of the sale proceeds and the penalty, or the collateral itself if you set `selfLiquidate`. |

## Taking part in a rollover
When a borrower calls `openRefinance`, a rate starts at the loan's blended APR and rises over 4 hours towards a cap set 4 points higher. You can accept at any moment the rate looks right, using a signed lend offer whose price is at or under the current rate. Alternatively, hand a keeper a rate limit so that a rollover you would have taken does not pass while you are offline.

## Cancelling offers

Use the "Your offers" table to cancel. On-chain, `cancel(nonce)` flips a single bit in your nonce bitmap, which invalidates all offers using that nonce, while `cancelWord` can retire as many as 256 nonces in one call. Any offer also lapses on its own once its expiry is reached.

## What Spatial charges

* A tenth of your interest, 10 percent, taken out of each payout.
* 5 percent of what parked capital earns, after that fee is enabled via the timelock.

## Practical tips

* If you want a bigger margin than the tier gives by default, set `maxLtvBps` lower than the tier maximum.
* Treat Tier C tokens individually, and before you lend against them, check DEX depth and read the auction history on Telemetry, which is where the public risk figures live.
* Choose expiries on standing offers that you are prepared to stand behind. An offer left open for 90 days at an old rate tends to get filled at the worst possible moment.
* If you are glad to end up owning the stock, set `selfLiquidate`. If you never want your slice sold over a weekend, set `noClosedMarketLiquidation` and build the gap risk into your APR.
