---
description: The three limits a tier sets on each loan against a token, the values in force today for each tier from A to D, and the process for moving a token from one tier to another.
---

# Tiers and LTV limits

A loan on Spatial takes three limits from the tier of the token backing it: the most that can be borrowed when the loan is written, the threshold at which a liquidation auction becomes possible, and the amount by which both are reduced while the underlying market is closed. Only two things set the tier: liquidity depth and oracle quality. The ticker, the issuer's brand and how popular a stock is play no part.

## Values in force

| Tier | Tier id in the contracts | Max LTV (open session) | Liquidation LTV | Haircut when the market is closed |
| --- | --- | --- | --- | --- |
| A | 1 | 55% | 70% | 10 pts |
| B | 2 | 45% | 60% | 10 pts |
| C | 3 | 30% | 45% | 15 pts |
| D | Not set up | Agreed with the lender; we suggest no more than 25% | None, with default only possible at maturity | Does not apply |

| Tier | Tokens included | Notes |
| --- | --- | --- |
| A | Index products and mega-caps: NVDA, AAPL, MSFT, SPY, QQQ | Their DEX pools are the deepest anywhere on Robinhood Chain |
| B | Large-cap single stocks, roughly the top hundred ranked by DEX liquidity | |
| C | Any other token with a live Chainlink feed | Lenders opt in token by token |
| D | Tokens without a live oracle, such as private-company tokens, where still offered (for example ones linked to OpenAI or SpaceX) | Nothing liquidates automatically. Planned for later, and limited to professional lenders |

{% hint style="info" %}
These are the live figures, taken from `RiskConfig.tierConfig`. Any change passes through the timelock, with its reasoning published and an entry added to the governance log. Read [Changing parameters](../transparency/parameter-changes.md) for the process.
{% endhint %}

The risk config will not accept a tier where max LTV is above liquidation LTV, where liquidation LTV is above 100 percent, or where the haircut exceeds the max LTV. As a result, no haircut can ever take a ceiling below zero.

## The role of each limit

**Max LTV** sets the ceiling when a loan is created. The hub takes the requested principal, divides it by the value of the collateral in basis points, and rejects anything over the ceiling with `LtvTooHigh`. On top of that, every lend offer has its own `maxLtvBps`, and the offer will only fill a request whose LTV is equal to or below it. Lenders can therefore demand lower leverage than the tier allows, but never higher.

**Liquidation LTV** marks the point where the health factor reaches 1.0. The gap between this and max LTV is the borrower's buffer, first before a warning is raised and then before an auction starts. The details are in [Health factor and liquidation auctions](../mechanics/health-and-auctions.md).

**The closed-market haircut** comes off both of the other limits whenever the feed is stale or the price comes from a closed session. Suppose a borrower opens a Tier A position at 55% on a Friday evening. Through the weekend its liquidation LTV sits at 60% instead of 70%. Origination follows the same logic, so the Tier A ceiling is 45% while the market is closed. During an open session a stale price is not haircut; instead, origination is blocked. See [pricing and sessions](../risk/pricing.md) for the full rules.

## Limits on exposure per token
Tiers control leverage loan by loan. Caps control total size token by token. `TokenConfig.exposureCap` limits the principal outstanding against a single collateral token across all loans together, and when a request would take the total past that limit, the hub reverts with `ExposureCapExceeded`. Caps begin low, sized against how deep the token's DEX liquidity has proved and how earlier auctions cleared, and they rise as the record of liquidations grows. Telemetry shows every token's cap alongside how much of it has been used.

## Moving between tiers

The tier is a single field inside a token's `TokenConfig` on the RiskConfig. Reviews weigh four things:

* how consistently the token's on-chain DEX depth has held up,
* how good the Chainlink feed is, covering update frequency, whether Data Streams include the token, and the deviation threshold,
* how closely earlier auction clearing prices tracked the oracle,
* issuer events and corporate actions.

As soon as the batch executes, new loans take the new tier. Existing loans keep both their rate and their term. Their liquidation LTV, though, is not locked in: every time the hub calculates a health factor it looks up the token's tier as it stands, so a reduced threshold applies to existing loans the moment the change goes through. The timelock delay serves as the warning period, and demotions are scheduled publicly so that borrowers have time to add collateral first.

## Tier D: loans without an oracle

Tier D is for tokens that lack a live price feed, in practice tokens tied to private companies. Lending against them is a bilateral deal agreed between professional parties. With no price, there is no basis for liquidation, so none happens automatically. The only recovery route is maturity: if the borrower still has not repaid when the grace window closes, the lenders receive the collateral. Tier D is planned, restricted to `LENDER_PROFESSIONAL`, and will be documented fully once its market launches.
