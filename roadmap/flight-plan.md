---
description: What Spatial runs on Robinhood Chain today, the markets whose first light is still ahead, and the proof each must show before it opens.
---

# Flight plan: what ships next

Spatial's first light was kept narrow on purpose. It opened with the most liquid collateral only, cautious LTVs, a publicly visible dollar cap per token and a public risk page from day one. Each later step is treated as its own mission. Every market listed here will have a separate first light, and none opens until the stage ahead of it has built a repayment and liquidation track record that justifies moving on.

## Live today

| Area | Current state |
| --- | --- |
| Collateral | Stock Tokens from Tier A (mega-caps and indices) and Tier B (large caps in single names), securing USDG loans on Robinhood Chain |
| Loan shape | Fixed rate; fixed term between 7 and 90 days (choose 7, 14, 30 or 90); syndicates of any size |
| Order book | Standing and targeted EIP-712 offers that cost nothing to sign or cancel, on a book anyone can view and export |
| Lifecycle | Top-ups, full or partial repayment, rollover auctions near maturity, and Dutch auction liquidation that can pay out in kind, with an opt-out while markets are closed |
| Parked capital | Unmatched USDG can be opted into a whitelisted Morpho vault and is withdrawn at origination just as it is needed |
| Risk controls | Exposure caps per token, caps on oracle moves, corporate action pauses, haircuts that follow the session, bounded floors during market closures, and a grace period once a sequencer outage ends |
| Participants | Lenders are professionals. Borrowers are verified professionals and businesses based in eligible jurisdictions |
| Openness | Contracts and services published as open source, oracle status, the governance log, Telemetry (public risk page and live readouts) and the Explorer, where every loan can be looked up |

## Markets awaiting first light

Each row is a market or capability not yet open, with its scope and the proof it is waiting on.

| First light | Covers | Prerequisite |
| --- | --- | --- |
| Tier C long tail | Every Stock Token with a live Chainlink feed, launched one market at a time, lenders opting in per token | Proven auction results and enough DEX depth across Tiers A and B |
| Tier D negotiated loans | Bilateral loans by professional lenders only, on tokens without a live oracle, defaulting only at maturity | Professional demand, plus legal review of each token |
| Senior and junior slices | Opt-in tranching within a syndicate, following the Centrifuge DROP/TIN model | A separate audit of the waterfall |
| Tokenized treasuries and funds | Bridged collateral priced at NAV (USDY, JTRSY, BUIDL-class funds) in isolated markets at 85% to 92% LTV | Bridge liquidity, plus the issuer allowlisting the escrow |
| Retail lenders | Enabling the `LENDER_RETAIL` role one jurisdiction at a time | A licensing route in each jurisdiction involved |
| Secondary market for slice tokens | Eligible lenders trading slices with each other on the platform | Nothing more; on-chain transfers are already restricted to eligible holders |
| Governance module | A module living on-chain replaces the foundation multisig for parameter control, retaining the timelock and the guarantee that repayment can never be paused | Design review and audit |
| More settlement assets | Markets in USDC or USDe beside USDG | Bridged liquidity confirmed, plus a vault to hold parked capital |

## How a cap goes up

Every collateral token carries a dollar exposure cap. Lifting it goes through the timelock like any other change, and the governance log records the reasoning next to the supporting evidence: DEX depth, the record of repayments, and auction prices compared with the oracle. See [tiers and LTV](../collateral/tiers-and-ltv.md) and [changing a parameter](../transparency/parameter-changes.md).

## Growing both sides

* **Supply comes first.** Credit funds, market makers and family offices already working on Robinhood Chain and Morpho keep standing offers on each token in Tiers A and B, so a borrower always finds a populated book.
* **Demand.** Holders of Stock Tokens who want USDG without selling, starting with traders on the chain's native DEX.
* **Distribution.** A dApp listing inside Robinhood Wallet, the Arbitrum and Chainlink builder programmes, and co-marketing with Paxos around USDG.
