---
description: How the Spatial contracts are tested, the audits, contests and bug bounty around them, the protections that stay active in production, and what the holders of governance keys can and cannot do.
---

# Assurance: testing, review and safeguards

Spatial borrows its approach from Morpho Blue: keep the immutable core small, prove it formally, have outsiders audit and attack it, and raise exposure only as quickly as the evidence supports.

## Governance keys and user funds

| Guarantee | How it holds |
| --- | --- |
| User money is out of reach | Nothing the multisig controls can touch escrowed collateral, principal due to lenders, or balances sitting in the reserve vault. |
| The core cannot be changed | `CreditHub`, `QuoteBook`, `SliceNote`, `LiquidationAuction` and `RolloverMarket` have neither a proxy nor an admin. Shipping a new version means deploying new contracts. |
| All other settings sit behind a timelock | Each adjustable value lives in `RiskConfig`, controlled by a multisig through a timelock. Changes enter the queue through `schedule`, tagged with a hash of the rationale published for them; once the delay has passed they run through `execute` and emit `ParamChanged`. Any batch not executed within 14 days expires. Once bootstrap is finished, the delay can never be set below one hour. |
| The pause is narrow | The only thing the guardian can do is call `setPaused`, which stops **originations and liquidations**, and the guardian cannot reverse a pause once raised. Repayment, adding collateral and releasing escrow after repayment cannot be paused by anyone. |
| Exposure grows in steps | Each collateral token has its own cap, set conservatively at first and raised as a record of liquidations and repayments builds up. |
| Builds can be reproduced | Anyone can rebuild the bytecode, and the source of every deployment is verified on Blockscout. |

To see how the timelock looks from a user's point of view, read [How parameters change](../transparency/parameter-changes.md).

## The testing stack

| Layer | Contents |
| --- | --- |
| Unit and integration | Written in Foundry: 139 tests across nine suites (Base, Invariants, Origination, Repayment, Reserve, LiquidationAuction, OracleGuard, RiskConfig, RolloverMarket). Together they exercise origination, repayment, liquidation, rollovers, pricing, parked capital and governance, including every path and every revert. [Build and deploy](build-and-deploy.md) lists the tests suite by suite. |
| Invariant and fuzz runs | A handler drives an open loan with randomised repayments, collateral top-ups and jumps forward in time, and after every step four bookkeeping rules are checked: the loan's principal equals the total of its slices, exposure for each collateral token equals the principal still outstanding, the hub holds at least the collateral its loans record, and the hub keeps no loan token once a transaction ends. Fuzzed cases add that a partial repayment of any size keeps the books consistent, that any split of lenders can be repaid in full, and that a liquidated loan releases all of its exposure. Focused tests cover interest that only ever grows, nonce and partial-fill accounting, how the health factor moves with price, and the auction price staying above its floor. |
| Mutation testing | Bugs are seeded into the source deliberately to prove the suite catches them. |
| Fork tests | The same suite runs against a fork of Robinhood Chain using the live Chainlink feeds, the live USDG and the live Morpho vaults. |
| Formal specifications | Properties proved over `CreditHub` and `QuoteBook`: principal cannot leave unless collateral is in escrow, collateral cannot leave while any debt remains, and no offer can be filled twice. |

The full stack runs on every change, and a release is only tagged when all of it passes.

## Independent review

* Each release of the contracts is **audited independently** before it reaches Robinhood Chain, and every audit is followed by a **public contest**. Reports will be linked from this page when they are published.
* A **bug bounty** is running, sized to the protocol's exposure. Responsibly disclosed findings are eligible from the day they are received.
* No release ships while a high or critical finding remains unresolved.

## Threats and responses

| Threat | Response |
| --- | --- |
| Offers that are forged or replayed | The EIP-712 domain commits to the chain ID and the hub's address; each maker cancels through a nonce bitmap; and fills are tallied per offer hash, so no capacity can be used twice |
| A dishonest relayer | Every offer is re-verified on chain. A relayer can hold an offer back but has no way to alter it |
| Price manipulation | Chainlink feeds and streams pass through session, staleness and pause checks that back each other up, along with a move cap. Prices always refer to the precise token sitting in escrow. Details: [pricing and sessions](../risk/pricing.md) |
| Collateral that is wrapped or priced by a derived rate | Refused. Each market reads only the feed for the token really sitting in escrow |
| Prices gapping over the weekend | Closed-market haircuts, auction floors with bounds, and opt-outs for individual lenders |
| A sequencer outage or censorship | A grace window after the sequencer comes back, and every call remains reachable through the L1 delayed inbox. Details: [the sequencer page](../risk/sequencer.md) |
| Failure of the reserve vault | Parking is opt-in, restricted to a single whitelisted vault, based on allowances, and within the scope of the audit. Details: [parked capital](../mechanics/parked-capital.md) |
| Stolen governance keys | The timelock leaves room for everyone to react, and the keys can neither move funds nor block a repayment |

## Reporting a vulnerability

Please email security@onspatial.org instead of opening a public issue. We acknowledge responsible disclosures within one business day, and they are eligible for the bounty.
