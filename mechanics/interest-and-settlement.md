---
description: How each slice accrues simple interest by the second, why a three-day floor applies, and how a full or partial repayment settles slices, collateral and the protocol fee.
---

# Accrual, repayment and settlement

## The amount a borrower owes

On Spatial, a loan's debt is two numbers added up over all of its slices: outstanding principal plus the interest that has built up. Calling `debtOf(loanId)` gives you both, projected forward to the present second, and it writes nothing to storage. A slice's rate is locked at the moment it is minted and stays put for life. There is no utilisation curve, nothing compounds and no one can adjust it later. When slices carry different APRs, each one accrues on its own.

## How interest accrues

Interest is linear, calculated by the `InterestMath` library:

```
interest = principal × aprBps × elapsed / (10_000 × 31_536_000)
```

Here `elapsed` is the number of seconds between the slice's `lastAccrual` stamp and `block.timestamp`, and the year is taken as 365 days. Multiplication comes first and division last, which means a single rounding step, always down. The protocol never measures time in block numbers, because on Arbitrum-stack chains those follow L1 blocks rather than the local clock.

## The three-day floor

When a slice is minted it stores a `minInterest` value: the amount its APR would produce on that principal across the risk config's `minInterestPeriod`, which is 3 days. The interest due is whichever is higher, accrued interest or that floor, minus any interest already settled. A borrower may repay on the first day, yet the lender is still paid for three. This stops standing offers being used as free same-day liquidity and rewards lenders for keeping them live.

## A single call to repay

Both partial and full repayment go through `repay(loanId, amount)`. The hub accrues all slices, totals what is owed and caps `amount` at that figure, so paying in excess of the debt is impossible. The hub pulls only that capped sum, in USDG, from whoever calls, and it settles interest ahead of principal.

Repayments are accepted for any loan in the Active or Defaulted state. That covers the full term, the grace period that follows maturity and the stretch after a rollover has failed. Repayment is blocked while a rollover or liquidation auction is under way, and after the loan has closed. See [the loan states page](loan-states.md) for the complete list.

`repay` sits outside every pause. The guardian's switches can halt new originations and liquidations, and nothing else.

### Paying off the whole debt

1. Interest is divided between slices according to each one's portion of the interest due; principal is divided by each one's portion of outstanding principal.
2. Every lender gets back principal and interest, minus the 10% interest share kept by the protocol, which goes to the FeeVault.
3. The hub burns each slice token and removes its slice record.
4. The loan moves to Repaid and the borrower gets the full escrow back.
5. `LoanRepaid(loanId, interest, principal)` is emitted.

If a lender's slice has the park flag set, their payout is deposited into the vault instead of their wallet. [Capital parked between fills](parked-capital.md) explains how.

### Paying off part of it

A payment below the full debt settles interest before principal, split between slices on the same basis as above. The collateral remains in escrow. Because the debt shrinks while the collateral does not, LTV falls and the health factor improves. The hub emits `LoanPartiallyRepaid(loanId, interest, principal)`. Once the health factor warning has triggered, a borrower has two remedies, and this is one of them; the other is `addCollateral`.

## Repaying via L1 when the sequencer is down

Should the sequencer halt or censor transactions, a borrower can submit `repay` through Arbitrum's L1 delayed inbox, which forces inclusion after 24 hours. Liquidations stay closed for one hour once the sequencer recovers, and that buffer is what makes this path usable. The [sequencer outage page](../risk/sequencer.md) has the details.

## Example

Three slices fund a loan of 20,000 USDG, and the borrower pays everything back on day 30.

| Principal | Slice | Interest over 30 days | APR |
| --- | --- | --- | --- |
| 5,000 | 1 | 34.93 | 8.50% |
| 10,000 | 2 | 73.15 | 8.90% |
| 5,000 | 3 | 36.99 | 9.00% |
| 20,000 | Total | 145.07 | |

The repayment is 20,145.07 USDG. Ten percent of the interest, 14.51 USDG, goes to the FeeVault. The three lenders receive 5,031.44, 10,065.83 and 5,033.29 respectively, each getting their principal back with 90% of the interest their slice earned. The same transaction releases the collateral from escrow.
