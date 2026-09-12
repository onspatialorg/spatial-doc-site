---
description: Spatial's fee schedule as stored in RiskConfig, who bears each fee, the moment the hub takes it, and where the money goes.
---

# Protocol fees

All fees are parameters in `RiskConfig` and are taken at the point where value moves. None is folded into the rates that lenders quote, and none is propped up by token emissions.

## Fee table

| Name | Payer | Current level | Parameter |
| --- | --- | --- | --- |
| Origination | Borrower | 0.25% of principal | `originationBps` = 25 |
| Interest share | Lender, deducted from interest | 10% of interest paid | `interestShareBps` = 1000 |
| Refinance | Borrower | 0.10% of principal | `refinanceBps` = 10 |
| Liquidation penalty | Borrower, out of auction proceeds | 3% of debt, with 1 point going to the keeper and 2 shared by lenders and the protocol | `penaltyBps` = 300 |
| Yield share on parked capital | Lender | 5% of vault yield, not active yet | `idleYieldShareBps` = 500 |

## Collection points

* **Origination:** deducted from principal when the loan is disbursed, and transferred to the FeeVault within that transaction.
* **Interest share:** taken from lender interest each time interest is paid, whether through repayment, a cleared rollover or interest recovered in a liquidation, at whatever rate applies at that moment.
* **Refinance:** charged at the moment a rollover auction clears, then added to the new principal.
* **Penalty:** paid out of auction proceeds. The keeper's point comes first; the remaining two follow once lender claims are met.
* **Yield share:** recorded, but no contract uses it until a timelocked change turns it on.

## How fees change

The only routes for changing a fee are `setFeeParams` and `setAuctionParams`, both behind the timelock. A batch is queued together with a rationale hash, sits through the delay, and is then executed, emitting `ParamChanged`. Past collections are never revisited, and origination and refinance fees are each charged a single time. The process is described on [How parameters change](../transparency/parameter-changes.md).

## Where fees go

The hub sends every fee to the `FeeVault` contract with an ordinary transfer. The money funds audits, day-to-day operations and the bug bounty. Withdrawals are restricted to the owner of `RiskConfig`, which is currently the multisig and will later be a governance module that decides how funds are spent.

## Rough scale

Suppose the average outstanding balance is 50M USDG, terms run 30 days and APRs sit at 8%:

* Origination: 50M turning over 12 times a year at 0.25% yields about 1.5M USDG.
* Interest share: 8% earned on 50M, of which the protocol keeps 10%, yields about 0.4M USDG.

That totals around 1.9M USDG per year, not counting liquidation income. [How Spatial earns](../resources/revenue.md) goes into more detail.
