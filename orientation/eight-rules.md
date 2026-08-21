---
description: The eight constraints every Spatial contract, service and screen must meet before release, and why each one exists.
---

# The eight design rules

Spatial avoids invention for its own sake. It reuses those pieces of on-chain lending which have held up under real capital and leaves out the parts that have not. Morpho Blue supplies isolated markets, oracle agnosticism and a deliberately tiny core. Blend and the NFTfi family supply off-chain signed offers with on-chain settlement, and Blend alone supplies the auction which moves a loan on to its next term. All of it answers to the eight rules on this page. If a change would break a rule, the change is abandoned and the rule stays.

## 1. Nothing is hidden

Every state change emits an event. The offer book is public as well, since anyone holding the signed messages can reconstruct it. Four views need no sign-in at all: the loan registry, which the platform calls the Explorer; the risk page, called Telemetry, showing live protocol-wide statistics and the parameters in force; oracle status; and the governance log. Contracts, relayer, indexer and keeper bots are open source with reproducible builds. Each commitment is spelled out on [the transparency checklist](../transparency/what-you-can-check.md).

## 2. Losses belong to whoever chose the exposure

A lender's slice is a separate position. Each collateral asset paired with a loan asset is a separate market configuration. When something goes wrong, whether through a weak asset, a poor borrower or a bad lending call, the party that picked that exposure absorbs the result. Any bad debt falls on the lender who priced the loan in question and on nobody besides.

## 3. Collateral exceeds debt and sits in escrow

Four tests apply to collateral: it is liquid, it has an oracle price, it was issued on this chain, and the escrow contract holds it. Anything failing them is excluded, which rules out uncollateralised loans and collateral held off-chain. When a loan fails, a liquidation auction settles it, and any address may start that auction. Courts play no part in recovery.

## 4. Price what is held, not a stand-in

The protocol values only the token sitting in escrow. Wrappers, derived exchange rates and substitute assets receive no price. In July 2026 Edel Finance lost funds after a wrapper's exchange rate was pushed to 78 times its underlying, while the oracle kept reporting accurate prices the whole time. Rule 4 alone would have prevented it.

## 5. Spatial holds nothing before a match

Until it is matched, a signed offer is just a message. Money moves at origination, under an approval the party gave the hub in advance, and at no other moment. Spatial never takes custody of unmatched lender capital. Lenders who want that capital earning can place it in a whitelisted Morpho vault, with an allowance they set themselves.

## 6. Rates are agreed, terms do not move

There is no utilisation curve in the protocol. A borrower states a maximum rate, each lender states a minimum, and the contract writes a loan where those ranges meet. Every term is fixed, somewhere from 7 to 90 days. When it runs out, a rollover auction takes over. Lenders know the exact day their money returns, and borrowers never face an abrupt repricing.

## 7. A fixed core with tunable edges

CreditHub, where every loan is originated, is one minimal, non-upgradeable contract modelled on Morpho Blue. Adjustable settings sit outside it. Fee rates, haircuts, LTV tiers, the vault whitelist and the approved attestation issuers all live in RiskConfig, a separate contract governed by a timelocked multisig, and each change emits an event. Each token's price feed and staleness bounds are set on OracleGuard by that same governance owner. Two actions can never be paused: repayment itself, and collateral coming back once a loan is repaid.

## 8. Designed around Robinhood Chain

Robinhood Chain depends on a single sequencer, prices equities 24/5 and flags the market's status alongside, and issues Stock Tokens without transfer restrictions of their own. Spatial is fitted to each of these. It allows a grace period once a sequencer outage ends and supports forced inclusion via L1. It applies session-based haircuts for as long as trading in the underlying is closed. And because the tokens have no allowlist, Spatial checks eligibility at its own edge.
