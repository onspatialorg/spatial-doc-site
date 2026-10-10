---
description: Each source of Spatial's income, the maths of what it adds up to for a large book, and which charges the protocol deliberately leaves out.
---

# Where the revenue comes from

## What stays free

The free parts come first, since they shape everything else.

* **No spread.** A lender sets a rate, a borrower takes it, and the contract settles at exactly that figure. There is no intermediary between them, so a peer-to-peer match gives the protocol no margin to capture.
* **No charge for order flow.** Anyone can read the public book of signed offers or export all of it.
* **No charge for data.** The indexer and the Explorer, Spatial's public loan explorer, are both open source and free to use.

Income comes instead from four points where the protocol does actual work: when a loan opens, when interest settles, when collateral goes to auction, and when a maturing loan rolls over.

## Fees at a glance

| Source | Payer | Initial rate |
| --- | --- | --- |
| Origination | The borrower | 0.25% of principal |
| Interest share | Lender, taken out of the payout | 10% of interest paid |
| Share of the liquidation penalty | A borrower in default, via the penalty | Of the three-point penalty, two points are split between the protocol and lenders, and one point goes to the keeper |
| Rollover | The borrower | 0.10% of principal |
| Share of yield on parked capital (roadmap) | Lender | 5% of yield |

[Fees the protocol charges](../mechanics/fees.md) lists the contract parameters that set these rates and the precise order of settlement.

## Why each party pays what it pays

Every fee falls on whoever gains from the work it covers.

* Origination is charged to the **borrower**, because the loan and the liquidity behind it exist at the borrower's request.
* The interest share comes from the **lender**. It scales with the return the market produced and is taken only when the lender receives payment. Principal is left alone.
* The rollover fee is charged to the **borrower**. Rollover auctions are there to take away the cliff at maturity, and that protection is the borrower's.
* The liquidation penalty is charged to a **borrower who has defaulted**. Splitting it three ways rewards the keeper for moving quickly and compensates lenders for the disruption.

## Projected income

Take an average outstanding book of 50M USDG, with loans averaging 30 days and 8% APR. With 30-day terms, the book turns over twelve times in a year.

| Item | Working | Yearly income (USDG) |
| --- | --- | --- |
| Origination | 50M at 0.25%, twelve turns | about 1.5M |
| Interest share | 10% of 8% on 50M | about 0.4M |
| **Total excluding liquidations** | | **about 1.9M** |

Liquidation penalties would add to this and are left out of the figures. Fixed costs, which cover day-to-day running, audits and the bug bounty, are covered once the outstanding book sits somewhere from 30M to 40M USDG.

## $SPATIAL

$SPATIAL is the protocol token and lives on Robinhood Chain, with its contract address listed on the website. Total supply is 1 billion, fixed, and there is no mint function in the contract. The token sits beside the fee model rather than replacing it. None of the income described here is distributed as token emissions, and emissions do not prop any of it up. The fees would be identical if the token did not exist. Some fee revenue goes towards buying back $SPATIAL.

## Where the money goes

Fees pay for audits, the bug bounty and the daily running of the service, and a share is reserved for buying back $SPATIAL. When control of parameters moves to the governance module, the split of that income will be decided through governance. [Foundation and operating company](../compliance/entities.md) describes the entities, and [How parameters change](../transparency/parameter-changes.md) covers how parameters get changed.
