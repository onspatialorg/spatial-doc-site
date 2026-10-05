---
description: The Borrow and Positions screens step by step, starting with the credential on your wallet and ending when your collateral is released.
---

# Taking a loan against your Stock Tokens

On Spatial you sign a request, lenders fill it, you settle the whole syndicate at once, and from then on you manage the loan on the Positions screen. You know the rate and the term before anything is binding. Your Stock Tokens are held in escrow by the hub, a contract that no custodian controls, and they are released as soon as the final USDG of debt is paid off.

## What you need first

* **A wallet on Robinhood Chain.** Robinhood Wallet works, as does any EVM wallet, and so does a smart account under 4337 or 7702.
* **Stock Tokens** you are prepared to pledge.
* **The `BORROWER` attestation** on that same wallet. Once the KYC provider has run its eligibility check it writes the attestation, and it then appears in Settings, in the "Attestations for this wallet" list.

At present the attestation goes to verified businesses and to individuals in an eligible jurisdiction who are classed as professional clients or as high-net-worth ones. Retail borrowers come in jurisdiction by jurisdiction, each one only once its consumer-credit rules are confirmed. The qualifying list lives on [Jurisdictions](../compliance/jurisdictions.md).

## Posting a request

You will find a single form on the Borrow screen, "Create a borrow request", with a "Fills and settlement" panel next to it.

1. **Connect your wallet.** The platform then picks up your Stock Token balances.
2. **Choose collateral.** Every token appears in the Collateral selector with its tier. The amount field has the oracle price beneath it, along with the current session: regular, extended or closed. Principal tells you the maximum you can draw at that tier's LTV, and Max APR suggests a rate taken from the live book for the term you pick.
3. **Complete the four fields:** Principal, Term, Max APR (%), Fill deadline (hours). Max APR is the ceiling on what you pay. Terms come in 7, 14, 30 or 90 days. The fill deadline is your launch window, meaning the time lenders get to fill you before the request expires. It starts at 24 hours.
4. **Check the live figures.** As you type, the form recalculates LTV, the opening Health factor and the Origination fee.
5. **Sign and post request.** What you sign is an EIP-712 message in the format the hub checks. There is no gas to pay for this step.
6. **Follow the fills.** Every open request in "Fills and settlement" has a coverage bar and a blended APR. The relayer matches standing and targeted offers the moment they come in, and lists each slice with its lender, the amount filled and its APR. Until coverage hits 100 percent you are not bound to anything.
7. **Settle.** One transaction sends the matched offers to `CreditHub`. Your collateral goes into escrow and you receive USDG, with the origination fee already taken off the principal. A smart account bundles the approval and the origination into one user operation. While the request is still open there is a Cancel button next to Settle.

## Managing an open loan

Open the Positions screen and look under "Loans you took". Each loan card gives you its Debt and its Accrued interest, its Health factor, how many slices it has, along with the blended rate, the term, a countdown to maturity and the oracle status of your collateral.

| Action | How | Result |
| --- | --- | --- |
| Add collateral | Enter it in "Add collateral" | Your health factor goes up. |
| Partial repayment | Put a smaller amount in "Repay" | Each slice's principal shrinks by the same proportion, and your health factor goes up. |
| Full repayment | Press Max in Repay | A single transaction clears principal and interest and hands back the escrow. |
| Rollover | Call `openRefinance` on the rollover auction ahead of maturity | A rate auction starts at your blended APR and rises until a fresh syndicate carries the loan into a new term. |

You can repay whenever you like. Interest, though, has a floor of three days, so even a loan closed within hours of opening is charged for three full days. The rollover is covered fully in [Maturity, grace and rollover](../mechanics/maturity-and-rollover.md).

## Notifications

Register alert channels and you are told when:

* your health factor falls below 1.10,
* the market is near its close and your closed-market health factor is close to the threshold,
* your maturity date approaches,
* lenders join a rollover auction you started,
* the oracle pauses for a corporate action, or the market itself is paused.

## When the term ends

At maturity a grace window of 24 hours begins, and you can still repay during it. Once it ends, a loan that was not repaid or rolled over is in default, and its collateral becomes eligible for a Dutch auction. The proceeds cover the debt and the penalty, and anything left over is returned to you.

## If the price drops

Once your health factor is under 1.0, any keeper is allowed to start the Dutch auction. The price begins above the oracle reading and steps down towards a floor. From the sale, the debt and a 3 percent penalty are paid, and the remainder goes back to you. Over the weekend, the closed-market haircut pulls liquidation LTV down and bounds the floor, so size your buffer for the market being shut. The full mechanics are on the [health and auctions page](../mechanics/health-and-auctions.md).

## Fees you pay

| Charge | Rate | Taken |
| --- | --- | --- |
| Origination | 0.25% of principal | On settlement, from the principal |
| Rollover | 0.10% of principal | Once a rollover auction clears |
| Penalty on liquidation | 3% of debt | Only when the loan is liquidated |

## Good practice

* Borrow comfortably under the tier maximum. The distance between max LTV and liquidation LTV is all the cushion you have.
* Check your closed-market health factor before the Friday close, not after the weekend.
* Keep some USDG on hand. Paying down part of the loan tends to be the cheapest way to raise your health factor.
* Stock splits and dividends are handled by the token's multiplier, so you never have to act on them.
