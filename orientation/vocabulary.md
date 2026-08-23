---
description: A glossary of the words used across the docs and the platform, each explained a single time and sorted by the part of the system it belongs to.
---

# Glossary

Financial terms here keep their standard industry names and meanings. On top of them, the platform uses five names of its own for screens and records; those come at the end, after the mechanics. Where a term corresponds to a contract, the contract is named.

## Requests, offers and matching

| Term | Definition |
| --- | --- |
| **Borrow request** | A borrower's ask for a loan, either posted on-chain or passed to a relayer. It specifies the collateral token and quantity, the principal wanted, the highest APR the borrower accepts, the term and a launch window. |
| **Offer** | A signed EIP-712 statement of the terms one side will accept. Lenders sign lend offers, while the borrower's side takes the form of the borrow request. Posting and withdrawing offers both cost nothing. |
| **Standing offer** | A lend offer not tied to any one request. Like a limit order sitting on a book, it sits against one collateral token or spans an entire tier. |
| **Targeted offer** | A lend offer written for exactly one borrow request. |
| **Origination** | The one transaction where CreditHub pairs offers with a request, escrows the collateral, mints the slice tokens and transfers each lender's principal across to the borrower. Where a page says settlement, it means this same step. |
| **Market** | The unit that isolates risk: a single collateral token against a single loan token, priced by that token's oracle and limited by the LTV settings of its tier. Events in one market never spill into another. |

## Loans and their participants

| Term | Definition |
| --- | --- |
| **Syndicate** | The group of lenders whose offers jointly filled a borrow request, each with a pro-rata slice. |
| **Slice** | The share of a single loan held by one lender, represented by a slice token. |
| **Slice token** | An ERC-721 minted per slice at origination by the SliceNote contract, in the collection "Spatial Slice Note" with symbol `SPSN`. It records principal, APR and start time and carries that slice's claim on the collateral. It can only be transferred to an address the permission registry recognises as a lender. |
| **Parked capital** | Sometimes called idle capital: USDG a lender has approved which no request has matched so far. For lenders who opt in, ReserveAdapter places it in a whitelisted Morpho Blue vault in the meantime. |
| **Keeper** | A bot anyone may run, with no permission needed, that starts auctions, raises alerts, bids in rollovers or shifts parked capital. |

## Valuation, sessions and loan health

| Term | Definition |
| --- | --- |
| **LTV** | Loan-to-value: principal divided by the collateral's value at the oracle price. |
| **Max LTV** | The highest LTV allowed when a new loan originates, set per tier and per session. |
| **Liquidation LTV** | The LTV beyond which a loan becomes liquidatable. |
| **Health factor (HF)** | In contract terms `value × liqLtv / debt`, meaning collateral value multiplied by liquidation LTV and divided by debt. Below 1.10 the platform warns the borrower; below 1.00 any address may start a liquidation. |
| **Session** | The oracle's report of the underlying equity market's state: regular hours, extended hours or closed. |
| **Closed-market haircut** | A reduction in the effective LTV limits that applies while the Chainlink feed reports a closed market or a stale price. |

## How loans end

| Term | Definition |
| --- | --- |
| **Grace window** | A 24 hour period after maturity during which repayment is still accepted. The loan counts as defaulted only once it has closed. |
| **Rollover auction** | The route by which a loan enters a fresh term. As maturity nears, the offered rate rises linearly from the existing rate toward a cap; the first lenders to accept take over the loan and the departing lenders are repaid. It runs on RolloverMarket and starts with a call to `openRefinance`. |
| **Dutch auction** | The sale of collateral from a defaulted or unhealthy loan. Pricing begins above the oracle quote and declines linearly to a floor, and the first buyer to accept the going price receives the collateral. It runs on LiquidationAuction. |

## Chain, assets and governance

| Term | Definition |
| --- | --- |
| **Stock Token** | An ERC-20 whose issuer is Robinhood Assets (Jersey) Ltd. Legally each one is a tokenized debt security, and shares kept in custody in the US back it one to one. |
| **`uiMultiplier()`** | The ERC-8056 view returning how many shares one Stock Token represents. Stock splits and reinvested dividends are reflected here. |
| **`oraclePaused()`** | A flag set around corporate actions, available from the token or from a wrapper around its Chainlink feed. While set, that token accepts neither new borrows nor liquidations. |
| **USDG** | The Paxos Global Dollar, the currency every loan is denominated and settled in. |
| **RiskConfig** | The timelocked contract holding every adjustable parameter. Each change made through it emits an event. |
| **Sequencer Uptime Feed** | A Chainlink feed reporting whether the L2 sequencer is up. Spatial reads it so that, after an outage, liquidations wait out a grace period. |
| **Delayed inbox** | An Arbitrum stack contract on L1 that forces a transaction into the chain after 24 hours, for times when the sequencer is offline or censoring. |

## Who may take part

| Term | Definition |
| --- | --- |
| **Permission registry** | PermissionRegistry checks attestation-backed roles at every point where eligibility counts: origination, acceptance of a rollover offer, and transfer of a slice token. |
| **Attestation** | An on-chain record, patterned on Ethereum Attestation Service attestations and issued by an approved KYC provider, confirming that a wallet meets a role's requirements. Besides an expiry date it records a jurisdiction class, and nothing in it identifies anyone personally. |

## Names used on the platform

A handful of screens and records on the platform have their own names. There are exactly five, and they appear in the platform's navigation and headings as well as throughout these docs.

| Term | Definition |
| --- | --- |
| **Explorer** | The public loan explorer at `/platform/explorer`, listing every loan, viewable without signing in. See [the Explorer guide](../guides/explorer.md). |
| **Telemetry** | Spatial's public risk page, together with its live figures: statistics across the protocol, exposure concentration by collateral token, and the parameters currently in effect. |
| **Flight log** | The event history of a single loan, covering origination, top-ups, repayments, rollover auctions and any liquidation, with each entry linking to its Blockscout transaction. |
| **Launch window** | A borrow request's fill deadline. After it passes, offers can no longer fill that request. |
| **First light** | The public launch of the platform, and thereafter the public opening of each additional market. |
