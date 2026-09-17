---
description: The asset classes queued after Stock Tokens, the settings they would launch with, and the onboarding process that keeps every new market isolated from the rest.
---

# Upcoming collateral

The hub contains nothing that only works for equities. A market consists of a collateral token with a price feed, a pairing to a loan token, a tier and a cap, so any token that Chainlink or its issuer can price could become one. Stock Tokens were first because they are issued on this chain and came with a broker's customers already holding them. Below is what follows and how new collateral is admitted.

## Treasuries and fund shares on chain

Tokenised US Treasuries are the largest RWA category after stablecoins, with roughly $16B on chain. Examples include USDY, JTRSY and BUIDL's share class, and they can arrive over LayerZero or CCIP. As collateral they behave as the opposite of a stock: volatility is close to zero, and the price is a NAV instead of a traded value.

They would launch with roughly these settings.

| Parameter | Planned setting |
| --- | --- |
| Origination max LTV | 85% to 92% |
| Price source | A NAV from the issuer or from Chainlink, with staleness limits set for a daily update |
| Liquidation LTV | A few points above the origination ceiling |
| Sessions | Not relevant: with no trading session there is no closed-market haircut |
| Transfers | Frequently permissioned under ERC-3643 or something similar; the escrow needs the issuer's allowlisting before launch |

## Robinhood's own funds

Should Robinhood issue tokenised funds on its chain, they pass through the usual gate: oracle, liquidity and bytecode reviews, followed by a tier set in a timelocked batch.

## Isolated markets

Each type of collateral has its own configuration: in `OracleGuard`, a dedicated feed paired with a loan token, and a dedicated tier and `exposureCap` in `RiskConfig`. Markets share nothing. Lender funds are not pooled, tokens do not cross-collateralise one another, and there is no shared bad-debt account. A problem with a bridged Treasury token stays within that token's loans, and an event affecting a Stock Token cannot reach a Treasury loan.

## Onboarding steps

1. Inspect the bytecode for any power to blacklist, freeze, pause or force transfers.
2. Assess the oracle: whether there is a feed, its update frequency, its deviation threshold, and its behaviour when paused.
3. Assess liquidity, meaning on-chain DEX depth or, for assets priced at NAV, the terms for redemption.
4. Publish the proposed tier and cap along with the reasoning, and leave it through the whole timelock.
5. Launch with a low cap, then lift it as data on repayments and liquidations accumulates.

## Collateral with compliance hooks

Tokens following ERC-3643 or ERC-7943 (uRWA) control their own holder lists. Accepting one as collateral means the hub must call `canTransfer` and `canReceive` both when collateral enters and when it leaves, so any restriction surfaces at origination and not when the borrower comes to repay. Today the hub moves collateral using ordinary ERC-20 transfers without those checks, and adding them is part of the work of launching a permissioned market. The [permission registry](../architecture/eligibility.md) is designed so that the token's rules and Spatial's rules can both apply, with neither having to trust the other.
