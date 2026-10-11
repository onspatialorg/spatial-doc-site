---
description: Spatial matches lenders and borrowers directly on Robinhood Chain for USDG loans secured by tokenized stocks, at a rate and term both sides fix up front, with contracts that never take custody.
---

# Spatial at a glance

<figure><img alt="Spatial logo mark" src=".gitbook/assets/logo-mark.png" width="168"><figcaption>Spatial's mark, a sphere of glass with folded layers of teal and aqua inside it.</figcaption></figure>

Spatial is a credit market with two sides and no pool in the middle. Borrowers bring Robinhood Stock Tokens and ask for USDG. Lenders decide which of those requests they want to fund and at what price. When the two agree, a contract escrows the collateral, pays out the principal and keeps the loan on record until it is closed. Everything else in these pages is about doing that safely.

## A thirty day loan, told plainly

Picture a holder of tokenized stock who needs dollars for a month and would rather not sell. On the other side sits a lender who wants a known return over that same month, backed by something they can value. Here is how Spatial brings them together.

1. **The ask.** The borrower posts a borrow request naming the collateral token and quantity, the principal they need, the highest APR they are prepared to pay, the term, and a launch window after which the request stops accepting fills.
2. **The answers.** Lenders reply with lend offers. Each one is signed off-chain with EIP-712 and states that lender's own rate.
3. **The fill.** When enough offers cover the request, the borrower submits them together to the hub in one transaction. CreditHub verifies each signature and each party's eligibility, takes the collateral into escrow, pays each lender's USDG across to the borrower, and issues every lender one slice token.
4. **The term.** Interest builds at the agreed rate from start to maturity.
5. **The end.** At maturity one of three things happens: the borrower pays back; a rollover auction passes the loan to fresh lenders at a climbing rate; or, after a default, the collateral is sold by Dutch auction.

Lenders waiting for a fill need not sit on cash. If they opt in, their USDG goes into an existing Morpho money market on the chain and stays there until the hub draws it into a loan.

## The market it serves

Private banks have sold this product for decades under the names Lombard loan and margin loan: credit secured by a client's securities. Spatial offers it on-chain. Collateral is Robinhood Stock Tokens, meaning tokenized AAPL, NVDA, SPY and many others, and every loan is in USDG.

No curve decides the price. A borrower names a ceiling, each lender names a floor, and wherever the two ranges meet, a loan is struck. A single lender can fund a request alone, or a syndicate can share it, with each member holding a slice.

Spatial never holds anybody's money. A lender's USDG sits in that lender's own wallet right up to origination. Collateral is locked in an escrow contract that cannot be changed and answers to no administrator. When a loan fails, that contract auctions the collateral on a falling Dutch price, with no court involved at any stage.

## Why a pool is the wrong shape here

Pools are excellent for deep, liquid collateral. They struggle with a single company's stock, a token whose DEX liquidity is thin, a weekend with the exchange closed, or a token representing a private company. Every one of those cases wants a price settled per loan and a term that actually ends. That slice of the market is where Spatial works.

| | Pooled RWA money market | Spatial |
| --- | --- | --- |
| Rate set by | A utilisation curve; it moves | Borrower and lenders; fixed for the full term |
| Loan ends | Only when someone closes it | At maturity, extendable through a rollover auction |
| Eligible collateral | A few liquid assets, each under a tight cap | Any Stock Token with an oracle price, long tail included |
| Risk borne by | All suppliers together | Only the lender or syndicate behind that loan |
| Bad debt hits | Every supplier | The lender who set that loan's price |
| Capital not yet lent | Earns whatever the pool pays | Waits in a Morpho vault for a match |
| Collateral recovery | Sold into DEX liquidity | Dutch auction with a bounded floor, or settlement in kind |

## Key facts

| Item | Detail |
| --- | --- |
| Network | Robinhood Chain, an Arbitrum Nitro L2 with chain ID 4663 |
| Loan currency | USDG (Paxos Global Dollar), for every loan |
| Accepted collateral | Robinhood Stock Tokens |
| Coming later | First the long tail of Stock Tokens, then tokenized treasuries and fund shares; see the [Flight plan](roadmap/flight-plan.md) |
| Loan structure | Rate and term both fixed; one request can be shared across as many lenders as needed |
| Custody | Spatial holds nothing. Principal only moves at origination, and collateral rests in immutable escrow |
| On default | A Dutch auction run by the contract, with no legal proceedings |

## What Spatial will not do

* Write a loan that is less than fully collateralised. Unsecured and undercollateralised credit are both ruled out entirely.
* Run pooled vaults of its own.
* Pay for lower rates with token emissions. $SPATIAL exists, but protocol income comes from explicit fees, set out in [How Spatial earns](resources/revenue.md).

## Why "Spatial"
Spatial is about position. Each loan has a fixed place: its rate and term are pinned at origination and stay put however the market moves. And each loan can be located by anyone, because every step lands on Robinhood Chain and the Explorer shows the full public record of every loan.

## Suggested paths through the docs

* **New here?** Read [the case for this market today](orientation/the-moment.md) and [the eight rules behind the design](orientation/eight-rules.md), keeping [the vocabulary](orientation/vocabulary.md) handy.
* **How it works.** Begin with [the mechanics tour](mechanics/tour.md), then [loan states](mechanics/loan-states.md), [how offers become loans](mechanics/offers-and-origination.md) and [health and auctions](mechanics/health-and-auctions.md).
* **Risk.** See [the register of risks](risk/register.md) and [the precedents behind the design](risk/precedents.md).
* **Integrating.** Look at the [system map](architecture/system-map.md), [the contracts](architecture/contracts.md) and [building and deploying](architecture/build-and-deploy.md).
* **Using the platform.** Follow the guides for [borrowers](guides/borrowing.md) and [lenders](guides/lending.md), and use [the Explorer guide](guides/explorer.md) to read loans that are not yours.
