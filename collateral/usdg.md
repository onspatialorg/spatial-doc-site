---
description: The reasons Spatial denominates all loans in USDG, the conditions for adding another loan currency, and the way the contracts handle the difference in decimals.
---

# USDG as the loan currency

Spatial uses a single loan currency throughout. Borrowers draw principal in USDG, interest builds up in USDG, fees are charged in USDG, and liquidation auctions settle in USDG. The token is Paxos's Global Dollar.

## Why USDG

* **Native to the chain.** USDG is minted directly on Robinhood Chain, so there is no bridge between any lender and any borrower.
* **The chain's established lending asset.** Robinhood Earn, the network's first lending product, uses USDG via Morpho, which meant the liquidity and integrations existed before Spatial launched.
* **Regulated under MiCA.** The protocol's main market is the EEA, so an EU-regulated stablecoin is important.
* **The same asset as idle capital.** Lender funds [waiting between fills](../mechanics/parked-capital.md) go into a whitelisted Morpho vault, and that vault holds USDG.

## How the contracts fix the currency

The on/off control is `RiskConfig.setLoanTokenEnabled`. Through `OracleGuard.configureFeed`, every collateral token is tied to a single loan token, and the hub refuses requests that specify anything else (`LoanTokenMismatch`). The loan currency is thus one of the things that defines a market, rather than something chosen per request.

## Handling the units

Stock Tokens use 18 decimals and USDG uses 6. The guard rescales the Chainlink answer and returns what one whole collateral token is worth, denominated in base units of the loan token. The hub then multiplies the escrowed quantity by this price and divides the result by ten raised to the collateral's decimals, a value it obtains from `OracleGuard.collateralDecimals`:

```
value = amount * price / 10^collateralDecimals
```

After this step, LTVs, health factors and interest are all calculated in loan-token units. The [assurance suite](../architecture/assurance.md) tests the arithmetic in `test/Invariants.t.sol`.

## Adding another currency

Since the loan token helps define a market, introducing a new one produces a separate, isolated market, and existing loans are unaffected. Two candidates are being looked at:

* USDe, which is already on the chain.
* USDC via CCTP, once its bridged liquidity is confirmed.

Neither will be switched on until three things are in place: verified liquidity, a review of the token's pause and blacklist powers, and a whitelisted vault where it can be parked. Turning it on is done through a timelocked batch, like every other parameter change.
