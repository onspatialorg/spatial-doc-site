---
description: The material risks Spatial runs, ordered by how severe they are, each matched to the control that addresses it, together with the self-imposed limits and the places where the safeguards stop.
---

# Risks and their controls

The whole design rests on two promises. Lenders are promised collateral worth more than what they are owed. Borrowers are promised that any liquidation runs by published rules, with no surprises. Every mechanism in Spatial exists to keep one promise or the other, and this register lists the things that could break them. The following pages go deeper on the biggest entries, one per page: [prices and sessions](pricing.md), [sequencer outages](sequencer.md), [issuer exposure](issuer.md), and [past failures](precedents.md) that informed the design.

## Self-imposed limits

Spatial keeps its scope narrow on purpose. Right now that means:

* Only Tier A or Tier B tokens qualify as collateral.
* Outstanding principal against each token is capped at a published dollar figure, and those caps begin low.
* Lenders must be professional. Borrowers must be verified businesses or verified professionals.
* Telemetry is the public risk page, open to anyone. It reports the spread of LTVs, how concentrated exposure is per token, any bad debt (the aim is none) and every single liquidation, with the price achieved set beside the oracle price at that moment.

Caps rise only once a record of liquidations and repayments has built up to justify them. The data comes first and growth follows.

## Every risk we track
| Risk | How severe | What answers it |
| --- | --- | --- |
| Reliance on the Stock Token issuer, a company in Jersey whose debt instrument includes rights to freeze | High | The tier LTVs are discounted to account for it. Before any market launches, its deployed bytecode is checked for freeze roles. Telemetry displays proof of reserve whenever a feed provides one. Each token gets a separate concentration cap. See [issuer exposure](issuer.md). |
| A jump in price between Friday's close and Monday's open, or over any other closed session | High | Haircuts that depend on the session, a bounded auction floor during closed hours, lender opt-outs and cautious tier LTVs. See [prices and sessions](pricing.md). |
| Thin DEX depth for an individual stock | High | Liquidated collateral is sold by Dutch auction rather than dumped on a DEX, and lenders may take it in kind instead. Every cap is set against the on-chain depth that has actually been seen. |
| A regulator recategorising what lenders hold, in any given jurisdiction | High | Only professional lenders may lend. The website blocks restricted regions by geo-fencing. Eligibility sits in a module that can be replaced, so stricter rules arrive without redeploying. Details in [Compliance at the boundary](../compliance/boundary.md). |
| The single sequencer halting or censoring | Medium | A grace window keyed to the uptime feed, plus a way to reach any function via Arbitrum's delayed inbox on Ethereum. See [sequencer outages](sequencer.md). |
| Oracle failure or oracle manipulation | Medium | Feeds and Streams from Chainlink are compared, every session has its own staleness bound, a cap on price moves triggers a pause, `oraclePaused` stops the market, and the price used is always that of the escrowed token itself, never of a wrapper. |
| A flaw in contract code | Medium | The immutable core is kept small and is formally verified, it has had two audits and a public contest, a bounty covers it, and caps go up in steps. Details in [Assurance](../architecture/assurance.md). |
| Not enough lenders, leaving the book thin | Medium | Standing offers, a return on idle capital, and professional lenders lined up ahead of each market launch. |
| Activity on the chain drifting to memecoins and away from RWAs | Medium | Focus on the holders of Stock Tokens and RWAs who are already there (over $100M in value) and expand in step with Robinhood's tokenisation plans. |
| Pool-based lending protocols starting to accept single-stock collateral | Low to Medium | A pool cannot replicate fixed terms, isolated positions, coverage of the long tail or syndication, because those are built into the structure. Spatial works with pooled markets instead of competing head on. |

## The edges of the safeguards

The design's limits get described as plainly as its protections.

* When a lender prices a slice badly, that slice loses money. This is how the market is meant to work, not a breakdown.
* When a borrower ignores the warnings, liquidation happens at the price the auction reaches. That price may fall below the oracle, and with the market shut it frequently does.
* Failure of the issuer is a real way to lose money. Lower LTVs reduce the loss but cannot eliminate it.
* Tier D is planned for tokens that have no live feed, and it has no liquidation at all. There, getting value back relies on the borrower repaying or on the lender taking the collateral at maturity.

## How a risk parameter gets changed

The risk parameters are: tier LTVs, haircuts for closed markets, the per-session staleness bounds, move caps, the auction curve's shape, how the liquidation penalty is divided, caps per token, and the whitelists for oracles and vaults. Every one of them follows the same path:

1. A proposer drafts the change, records the reasoning, and tests it against historical prices wherever that history is available.
2. The proposal stays public for the full length of the timelock.
3. Execution happens on chain, and an event is emitted.
4. The change is entered in the public governance log.

The full procedure is laid out in [How parameters change](../transparency/parameter-changes.md).
