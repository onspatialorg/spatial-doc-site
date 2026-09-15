---
description: Why a Robinhood Stock Token works as loan collateral on Spatial, and the two issuer-side risks the protocol builds into its pricing.
---

# Stock Tokens as loan collateral

In legal terms, each Stock Token is debt, not equity. Robinhood Assets (Jersey) Ltd is the issuer, and the token has a value that follows one share kept in US custody. Whoever holds the token has a claim against the issuer; the share itself is never theirs. That legal structure, together with three technical properties, is what allows Spatial to escrow the token and lend USDG on top of it.

## What the hub depends on

1. **A standard ERC-20.** The token uses 18 decimals, moves freely, and adds the ERC-8056 interface to handle corporate actions. Any wallet can hold one, and any contract can lock one in escrow.
2. **One-to-one backing.** For each token in circulation, a share sits in custody.
3. **Chainlink pricing 24/5.** Data Feeds publish through the trading week, Data Streams carry a market-status flag, and the quoted price already includes the ERC-8056 multiplier.
4. **Distribution through a regulated broker.** Sales reach over 120 countries, so the collateral had holders well before a lending market existed for it.

These properties explain why the protocol runs on Robinhood Chain, the network that issues the tokens. More on that in [Why Robinhood Chain](../chain/the-case-for-robinhood-chain.md).

## Quick facts

| Property | Detail |
| --- | --- |
| Issuer | Robinhood Assets (Jersey) Ltd |
| Legal form | Tokenised debt, not stock. Its claim follows the underlying share, which never leaves the custodian. |
| Backing | Each token has one share behind it, custodied in the US |
| Standard | ERC-20 at 18 decimals, plus the ERC-8056 extension for corporate actions |
| Reach | Over 120 countries |
| Restrictions enforced on chain | None documented. There is no allowlist, transfer hook or freeze function. Robinhood screens eligibility inside its app and at issuance and redemption, both of which require KYC. |
| Price source | Chainlink Data Feeds (24/5) plus Data Streams with market status; the multiplier is part of the price |

## Corporate actions under ERC-8056

Each token exposes `uiMultiplier()`, which says how many underlying shares one token represents. A dividend is reinvested, so the multiplier goes up. A stock split adjusts the same number accordingly. Nothing else changes.

For collateral held in escrow, that design helps the borrower in three ways.

* Dividends stay inside the collateral. The escrowed tokens are simply worth more, and no separate dividend token exists for anyone to claim, hold or argue over.
* Splits do not disturb the loan. Since Chainlink includes the multiplier in its price, the collateral keeps its correct value through the event and no one has to step in.
* Positions can be shown in shares. `OracleGuard` reads `uiMultiplier()` alongside the price and includes it in the quote it returns, so the Explorer can present an escrowed position both as tokens and as shares.

While a split, merger or similar event is processed, Chainlink sets `oraclePaused()` on the feed. Until that flag clears, the hub turns away new originations against the token and the liquidation auction will not open. Repayment carries on as normal: a paused feed never prevents a borrower from paying down the loan and taking collateral back.

## Eligibility is checked at the edge

The token has no transfer logic, so it can end up in any wallet, even one that Robinhood's own screening would have rejected. Spatial does not lean on the token to handle this. Instead the [permission registry](../architecture/eligibility.md) is checked wherever someone enters the protocol: the borrower at each origination, any relayer submitting for a borrower, every lender whose offer is filled, each bidder in a liquidation auction, and any recipient of a slice token transfer.

If Stock Tokens one day add ERC-3643 or ERC-7943 (uRWA) hooks, a market could only open once the issuer had allowlisted the escrow, and the hub would have to use hook-aware transfers. At present it uses ordinary ERC-20 calls to move tokens. Permissioned tokens are discussed further in [Upcoming collateral](next-markets.md).

## Issuer risk, in two parts

A loan backed by a Stock Token carries some risk unrelated to the share price. Spatial does not ignore it; both parts are priced in.

**Issuer credit.** A token holder has the Jersey issuer, Robinhood Assets (Jersey) Ltd, as counterparty. If it failed, holders would recover whatever the custodian returned, not the trading value of the share. Tier LTVs are already set with that case in mind, and Telemetry displays the issuer's reserve status wherever a proof-of-reserve feed exists.

**The right to freeze.** Under its terms, the issuer may freeze, suspend or otherwise restrict tokens in specified circumstances, although the public documentation shows no freeze function on chain. Ahead of any Stock Token market opening, the deployed bytecode is inspected for any blacklist, forced-transfer power, or role able to pause or freeze, and the findings are published on the risk page. Any such role is treated as tail risk and reflected in both the token's cap and its tier LTV.

Both risks are covered in depth on [The Stock Token issuer](../risk/issuer.md).

## Tokens accepted today

Tier A holds five tokenised names: NVDA, AAPL and MSFT, plus the SPY and QQQ funds. For the tier and cap of every token, see [Tiers and LTV limits](tiers-and-ltv.md).
