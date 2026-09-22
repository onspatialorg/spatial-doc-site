---
description: Holding a Stock Token means holding a debt claim on the company that issued it, Robinhood Assets (Jersey) Ltd, and not the underlying share. Here that exposure is split into its components, with where each one is priced, monitored and limited.
---

# Issuer exposure in Stock Tokens

Every Spatial loan contains a gap between what is held as collateral and the stock that collateral follows. A Stock Token is a tokenised debt security. Its issuer is Robinhood Assets (Jersey) Ltd, and it refers to a share held in custody in the United States. Spatial handles that gap as a distinct risk, with its own controls kept apart from the market-risk controls described on [the pricing page](pricing.md).

## The exposure, in four pieces

**Credit.** If the issuer became insolvent, token holders would be creditors with a claim on the custodied shares. Recovery might be partial, delayed, or both.

**Freezes and restrictions.** The issuer's terms allow it, in certain specified situations, to suspend, freeze or restrict a token. Were that applied to tokens held in escrow, the collateral would be stuck.

**Redemption.** Redeeming directly through the issuer happens in the primary market and requires KYC. Spatial does not rely on it at any point. Value is recovered through prices reached in secondary-market auctions.

**Regulatory action.** A regulator could compel the issuer to change its terms, remove countries from its eligible list, or halt issuance altogether.

## The lender's real position

Behind a Stock Token loan stands a claim on a regulated broker's custody arrangement rather than the share itself. A lender who prefers to finish with the stock instead of USDG can switch on `selfLiquidate` and take the collateral in kind, issuer claim included. The tier LTVs and the caps are, between them, the protocol's view of how heavily that claim ought to be discounted. A lender with a more cautious view can put a lower `maxLtvBps` in their offer.

## The controls

### A discount built into tier LTVs

Lending against the actual share, a prime broker might in principle go to 80% or higher. Tier A is capped at 55%. Most of that difference reflects discounts for liquidity and for issuer risk, set so that a sizeable haircut on the issuer can be absorbed before any slice takes a loss. Every tier's figures appear in [the tier and LTV table](../collateral/tiers-and-ltv.md).

### Reading the bytecode ahead of First light

According to the published Stock Token documentation, there is no freeze function on chain. Spatial checks rather than relying on that. Ahead of First light for each market, the deployed bytecode of the token is examined for any role able to pause, freeze, blacklist or force a transfer. Telemetry, the public risk page, shows the outcome alongside that market. Should any such role exist, the token's tier LTV and exposure cap are set to account for it.

### Proof of reserve shown on Telemetry

If a Proof-of-Reserve feed from Chainlink, or a comparable attestation, backs the shares in custody, Telemetry displays it beside the relevant Stock Token market. Where no such feed exists, the market states that clearly instead of showing an empty field.

### Limits per token and across the issuer

A cap called `exposureCap`, held in each token's configuration, sets the most principal that can be outstanding against that token. All Stock Tokens come from the same issuer, so Telemetry also reports a single combined figure: the protocol's total exposure to Robinhood Assets (Jersey) Ltd. A cap across the whole protocol on that total is being considered as a future parameter on the roadmap.

### No contagion between markets

Each market stands alone. An issuer event that hits a single Stock Token, or every Stock Token together, cannot spread to a market collateralised by a bridged treasury token.
