---
description: How a group of lenders funds a single loan, the data a slice token holds, and the way repayments, rollover funds and auction proceeds are split between slices.
---

# Syndicates and slice tokens

On Spatial, a loan with a single lender is still a syndicate. One or several offers fill a request, every fill turns into a slice, and every slice retains the APR chosen by the lender behind it. Syndication is not an add-on feature. It is simply the way loans are assembled.

## Anatomy of a slice

For every fill, `originate` records a slice and mints a single token from `SliceNote`, an ERC-721 collection named "Spatial Slice Note" with the symbol `SPSN`. The slice id is identical to the token id, and `SliceCreated` carries both. If two offers from the same lender both fill one request, that lender ends up with two slices.

Each slice record stores:

* the loan id, which also determines the collateral and loan tokens
* the lender
* principal still outstanding
* the slice's own `aprBps`
* the minimum interest due no matter how soon the loan ends
* interest booked so far, interest paid so far, and when it last accrued
* the three flag bits copied from the offer: park idle, self-liquidate, and no liquidation while markets are closed

Interest runs at each slice's own rate, and every payout, whether from repayment, rollover or auction, is calculated per slice. The risk config limits a loan to 32 slices, because repayment and liquidation loop over every one of them and a syndicate with no cap could grow beyond what fits in a block.

## Transferring a slice

Slice tokens can be transferred, provided the receiving address passes the lender check in the `PermissionRegistry`. The token contract then notifies the hub, which redirects future payouts to the new holder and emits `SliceTransferred`. With that single rule in place, loan positions can change hands while eligibility is preserved.

## The case for syndicating every loan

* **Big loans from small tickets.** A 200,000 USDG borrow against tokenised NVDA can be funded by twenty lenders putting in 10,000 USDG each.
* **Depth without a giant lender.** How much can be borrowed depends on the offers sitting in the book, not on any one balance sheet.
* **Risk that stays put.** Spatial has neither a pool nor a common bad-debt fund. A lender who priced a slice at 8.5% owns that slice, with its interest and any shortfall, and carries nothing else.

## Splitting the money

Write the principals of the slices as \(P_1\) to \(P_n\), with total \(P\).

* Interest on slice \(i\) grows linearly, by the second, with a floor equal to one minimum interest period:

  ```
  elapsed × apr_i × P_i / 365 days
  ```

* Repayments settle interest before principal. The interest portion is split according to how much interest each slice is owed, and the principal portion according to how much principal each slice still has outstanding, so a partial payment of \(R\) reduces slice \(i\) by \(R \times P_i / P\). Every lender gets principal plus interest after the protocol's interest share is deducted.
* When a rollover clears, the outgoing slices are repaid in full out of the incoming money and then retired, while incoming lenders receive fresh slices.
* Proceeds from an auction follow a waterfall. The keeper's penalty cut comes first, followed by the amounts lenders are owed, then the lenders' cut of the penalty, then the protocol's cut, and finally any surplus, along with unsold collateral, to the borrower. Lenders divide what reaches them in proportion to each slice's claim (principal and interest owed) against the total debt, which caps every slice at what it is owed plus its portion of the penalty. Any shortfall remains with the slices that incurred it.

What the borrower actually pays is the slice rates averaged with principal as the weight.

## All or nothing

A loan either originates in full or does not originate, and the offers have to add up to the request exactly. Coming up short reverts with `UnderFilled`, and an offer beyond complete coverage reverts with `OverFilled`. When there is not enough depth in the book for a request, nothing is written, and the borrower is free to ask for less or to wait until more offers arrive. Adding new slices to a loan that is already running is planned on the roadmap.

## Example: twenty lenders in a single call

Say a borrower requests 200,000 USDG. They post 2,500 NVDA tokens as collateral, worth roughly 441,000 USDG, putting LTV at 45.4%. The relayer assembles twenty standing offers of 10,000 USDG each, with rates spread from 8.2% to 9.0%. One `originate` call locks the 2,500 NVDA in escrow, takes 10,000 USDG from each lender (pulling from the Morpho vault for any lender whose capital is parked), and mints twenty slice tokens. After the origination fee the borrower gets 199,500 USDG.

Suppose the blended rate is 8.6% and the borrower repays on day 30. Interest comes to about 1,414 USDG, split over the twenty slices according to principal and each slice's own APR, with the protocol taking a 10% share of interest.

## Coming next: senior and junior tranches

The upcoming protocol release brings tranching within a syndicate, modelled on the DROP/TIN structure used by Centrifuge. Lenders who choose the junior slice absorb losses first if an auction comes up short, and earn a higher rate for doing so, while senior slices get paid first. The choice is made offer by offer, so lenders who never opt in notice no change. The new waterfall will receive its own separate audit before it goes live.
