---
description: How ReserveAdapter lets a lender's unmatched USDG earn yield in a whitelisted Morpho vault, and how the hub withdraws it inside the same transaction that fills an offer.
---

# Parking idle capital

Standing offers depend on the funds being available the instant a borrower turns up, yet USDG sitting in a wallet earns no return. Spatial borrows the approach Morpho Optimizer took on top of Aave and Compound: capital that has not been matched earns the pool rate, and once matched it earns the peer-to-peer rate instead.

## How parking works

1. A lender turns on the `parkIdle` flag for their offers and gives `ReserveAdapter` a USDG allowance.
2. Next they call `deposit(assets)`. The adapter deposits the USDG in the approved Morpho Blue USDG vault, an ERC-4626 vault deployed on Robinhood Chain, and credits the lender with the minted vault shares, tracked in its `shares` mapping. To the vault, the adapter is the depositor, while `balanceOf(lender)` shows what each lender has parked, denominated in USDG.
3. The relayer treats that parked balance as capacity backing the lender's standing offers.
4. Whenever an offer fills, or an acceptance is made in a rollover auction, the hub calls `withdrawFor` for precisely the amount of the fill. Pulling from the vault, creating the loan and paying out the borrower all happen within a single transaction.
5. When a park-flagged slice is repaid or liquidated, the hub pays it via `depositFor`, sending the proceeds directly back into the vault.

Lenders exit through `withdraw(assets, to)`. The alternative, `withdrawAll(to)`, redeems all their shares and closes the position without leaving dust behind.

## Approved vaults

Deposits can go only to a vault that has been approved with `setVaultWhitelisted` on `RiskConfig`. Today one vault is on the list: a well-established USDG vault managed by a conservative curator. Approving another vault requires a timelocked change, and the reasoning behind it is made public; [How parameters change](../transparency/parameter-changes.md) describes the process. The adapter's vault address cannot be altered, and the hub's adapter is locked in once it has been connected, so parked funds cannot be rerouted without anyone noticing.

Each deposit checks the whitelist. Delisting a vault stops fresh capital going in but leaves all withdrawal routes working, so funds are never trapped when a vault is retired. If a park-flagged slice is repaid after its vault has been delisted, the adapter sends the funds straight to the wallet of the lender and emits `ParkingSkipped`.

## Risks a parking lender accepts

Parking is optional because it brings risks the core protocol does not carry.

* **Vault exposure.** Whatever happens in the Morpho markets the vault allocates to, and whatever its curator decides, is the lender's risk.
* **Liquidity on withdrawal.** When a match comes in and the vault is unable to pay out, the fill reverts. The relayer monitors vault liquidity and never proposes a slice that the vault could not fund.
* **Extra code.** Origination passes through one additional contract. It is compact, its only permitted asset movements are into the vault, out to the lender, or to a destination the hub names, and it is included in every audit.

## Fee on vault yield

`RiskConfig` holds an `idleYieldShareBps` value of 500, meaning a 5% cut of vault yield, but at present no contract reads it. Parking is free until a timelocked change activates it. The [fees page](fees.md) lists it as a scheduled fee.

## What the adapter cannot do

* Send capital anywhere besides the approved vault, the lender, or the fill address the hub specifies.
* Rehypothecate the vault shares it holds.
* Switch to another vault. Using a new vault requires a fresh whitelist entry that goes through the timelock and is recorded by `ParamChanged`.
