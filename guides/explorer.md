---
description: A guide to the Explorer, Spatial's public record of all loans, and to the Telemetry, oracle, book and governance views that sit alongside it.
---

# Using the Explorer

The Explorer lives at `/platform/explorer`. You do not need to sign in or connect a wallet. It holds every loan Spatial has ever originated, and each event on each loan links to the transaction that produced it on-chain. This is where the transparency commitments can actually be tested by anyone who reads them.

## Headline figures

Across the top you see four numbers: total originated alongside the number of loans, outstanding principal alongside the number still active, participants split by side (lenders, borrowers), and bad debt next to the count of liquidations.

## All loans

The "All loans" table covers the full set, and its status filter offers active, refinancing and liquidating as well as repaid, settled and defaulted. For each loan it gives the borrower address exactly as recorded on-chain, the collateral, principal, blended APR, health factor, slice count, maturity and status.

Click any row to open the complete record:

* the collateral in token units and in shares, converted through the ERC-8056 multiplier,
* the oracle price at origination shown next to the current one,
* principal, blended APR, interest accrued to date, term and maturity,
* health factor and current session, plus the health factor the loan would have with the closed-market haircut applied,
* **Syndicate**: each slice with its lender's address, its principal and APR, and which flags it carries (in-kind, no-closed, idle),
* **Flight log**: every event on the loan in sequence, starting at origination and running through any top-ups, rollover auctions, partial repayments and liquidation, with a Blockscout link for each.

## Telemetry

The loan record is about one position. Telemetry is about the whole book, and shows the risk figures:

* How LTV is distributed over active loans.
* Concentration per token, plus total exposure to the Stock Token issuer. In the "Concentration by token" panel, each token's outstanding principal is plotted against its exposure cap.
* The bad debt ledger, which is meant to read zero.
* Every liquidation, setting the price the auction realised beside the oracle price at that moment.
* Results of rollovers, with the clearing rate and the time each took.

## Oracles

For each token you see the current Chainlink price and session, `updatedAt` together with how stale it is compared with the bound, the value of `uiMultiplier()`, the `oraclePaused()` flag, sequencer uptime, and any market pause in effect along with why it was applied.

## Book

A mirror of the relayer's open requests and standing offers, which you can export in full.

## Governance

Each `RiskConfig` proposal and each execution, listing the parameter, its old and new values, the stated reason, when the timelock began and when the change took effect. The "In force" panel lists current settings: the available terms, the origination fee and interest share, the refinance fee, the grace window, the minimum interest period, the penalty, where auctions start and where they bottom out, the cap on refinancing, the chain, and the address of `CreditHub`.

## How it is built

The front-end is built in Next.js and reads from the public indexer, and the code for each is open. No figure comes from a private database. Every number derives from chain events and signed messages alone, so if anyone else runs their own indexer and explorer, they will see the same results.

## Why it exists

A lending market earns only as much trust as an outside party can confirm for themselves. Before posting an offer, a lender can study the liquidation history. A borrower can see the rates others actually paid. Anybody can check that the live parameters match the announced ones. [What anyone can check](../transparency/what-you-can-check.md) sets out every commitment.
