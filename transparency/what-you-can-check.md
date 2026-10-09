---
description: Eight facts about Spatial that anyone can confirm using public data only, and the contract or service that guarantees each.
---

# Verifying Spatial yourself

Solvency is what Spatial commits to first. Its second commitment is that you never have to take the first on faith. Anyone outside the team can reproduce every number the platform shows from public data, and that is guaranteed by contracts and open services rather than by a written policy.

Two pages deliver this. Every loan is visible on the Explorer at /platform/explorer, Spatial's public view of its loans. Telemetry is the public risk page with its live readouts of statistics, concentration and current parameters.

## The commitments and their enforcement

| # | What is promised | How it shows up | What guarantees it |
| --- | --- | --- | --- |
| 1 | Every state change produces an event | Originations, repayments, rollovers, auctions opening or settling, and parameter changes all emit events, and the Explorer links each to its Blockscout transaction | `CreditHub`, `LiquidationAuction`, `RolloverMarket` and `RiskConfig` emit on each write, and the indexer and Explorer publish what they emit |
| 2 | Anyone can read the loan book | Each live loan shows collateral, LTV, rate and term, syndicate size, health and the oracle price at origination | The loan and slice records held in `CreditHub`, which the indexer feeds into the Explorer |
| 3 | Risk statistics are open | LTV distribution across the protocol, concentration by token and by issuer, a bad debt ledger expected to show zero, and each liquidation's realised price set against the oracle | Telemetry, which the indexer calculates from on-chain events |
| 4 | Oracle and reserve data can be seen | For each token: its pause flag, multiplier, session, staleness and live Chainlink price, plus issuer reserves where a feed is available | `OracleGuard.quote()` plus the proof-of-reserve feed published by the issuer |
| 5 | Parameter changes appear in the log before they apply | Each proposal and execution is logged with its reasoning before it takes effect | `RiskConfig`, where `OperationScheduled` includes the rationale hash and `ParamChanged` is emitted on execution |
| 6 | All code is published | Contracts, relayer, indexer and keeper bots, each with a reproducible build | Permissively licensed public repositories, with bytecode verified on Blockscout |
| 7 | The order flow is visible | Anyone who runs their own relayer sees exactly the same offers, and the whole off-chain book can be exported | Offers are EIP-712 messages in the `QuoteBook` domain, and the relayer exports the book |
| 8 | No funds are held ahead of a match | Money moves only at origination, and no one can pause repayment or the withdrawal that follows it | `CreditHub.originate` draws funds through Permit2 or a prior approval, while `repay` and escrow release have no pause control |

## Everything comes from two sources

The complete input set is on-chain events plus signed offers. With only those, an outsider can recalculate every figure that Telemetry or the Explorer displays. Neither page relies on a private database. If Spatial Labs disappeared tomorrow, running the open-source indexer against the chain would rebuild both pages exactly.

## The only private information

Just two things are kept out of view:

* Personal data behind an attestation. The KYC provider keeps it, and it is never written on-chain.
* The real person or entity behind a wallet. Addresses appear only as addresses.

Every other detail of a loan is deliberately public.

## Scheduled risk reporting

Alongside the live readouts, the next item on the transparency roadmap is a periodic risk report. It will cover liquidations, open exposure, auction results compared with oracle prices, parameter changes and incidents, each linked to the on-chain evidence that supports it.
