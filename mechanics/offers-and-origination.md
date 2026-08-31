---
description: Field by field coverage of the LendOffer and BorrowRequest messages, how the nonce bitmap and fill ledger behave, what a relayer proposes, and the precise sequence of checks run inside originate.
---

# Signed offers, signed requests and how a loan originates

A loan is born when two kinds of signed message come together in a single transaction. The lender's side is a `LendOffer`, the borrower's side is a `BorrowRequest`, and the hub checks both before any token changes hands. Producing a signature costs nothing, withdrawing one takes a single inexpensive call, and settlement validates each message before it spends a cent. The same approach underpins Blend, NFTfi and exchanges modelled on Seaport: lenders get a limit-order book in which posting or pulling an order is gasless.

## Domain for signatures

Each message is EIP-712 typed data. Its domain has the name `Spatial`, version `1` and chain ID 4663, with the hub as verifying contract. `LendOffer` and `BorrowRequest` are the primary types. If the chain ever forks, the hub recalculates its domain separator so that signatures made on the other branch are rejected. A wallet can obtain the exact digest to sign from `offerHash` or `requestHash`.

## Anatomy of a lend offer

```solidity
struct LendOffer {
    address maker;           // lender whose signature is verified
    Side    side;            // encoded as uint8; only 0 = Lend can be filled
    address collateralToken; // a specific token, or address(uint160(tier)) for a whole tier
    address loanToken;       // USDG
    uint256 principalMin;    // minimum fill size, waived when the fill empties the offer
    uint256 principalMax;    // ceiling on lending summed over every fill
    uint16  aprBps;          // fixed rate in basis points
    uint16  maxLtvBps;       // most aggressive request LTV the lender accepts
    uint32  termSeconds;     // has to match the request's term
    uint40  expiry;          // final second of validity
    uint256 nonce;           // slot in the maker's cancellation bitmap
    bytes32 salt;            // keeps each digest distinct
    bytes32 requestId;       // zero, one request's digest, or a rollover key
    uint8   flags;           // flag bits, described below
}
```

`maxLtvBps` limits only what this maker will fund. The hub checks the tier ceiling on its own, so setting the field higher than that ceiling has no effect. Treat `nonce` as a handle for cancelling, never as a counter. Filling an offer does not touch it, and a maker can give many offers the same nonce so that one call retires them all.

### Targeted versus standing offers

Where an offer may be spent is decided by `requestId`. If it holds a request's digest, the offer is targeted and can fill that request alone. If it holds zero, the offer is standing: `collateralToken` then names a single token or the sentinel address of a tier, and any request with compatible terms may draw on it again and again until `principalMax` runs out. Setting it to `QuoteHash.rolloverKey(loanId)` instead ties the offer to the rollover of one particular loan. Depth in the book comes mostly from standing offers. A professional lender who posts one against every Tier A token gets matched with a stream of borrowers and never has to sign again.

### Flag bits

`FLAG_SELF_LIQUIDATE` (bit 0): if the loan is liquidated, this lender takes payment in collateral, valued at the guard's price, ahead of any auction sale, up to the slice's pro rata portion of the escrow. It suits lenders who would rather end up holding the stock.

`FLAG_NO_CLOSED_MARKET_LIQUIDATION` (bit 1): should an auction start while equity markets are shut, this slice is lifted into a new loan that carries on, so the lender accepts gap risk rather than a sale over the weekend. When all slices set the bit, a closed-market auction cannot start at all.

`FLAG_PARK_IDLE` (bit 2): USDG that has not been matched sits in the whitelisted Morpho vault reached through the `ReserveAdapter`. At origination, or when a rollover acceptance happens, the hub pulls out precisely the fill amount, and repayments flow back into the vault. More on this in [Capital parked between fills](parked-capital.md).

## Anatomy of a borrow request

The request specifies the `borrower`; the `collateralToken` and `collateralAmount` destined for escrow; the `loanToken` and `principal`; a `maxAprBps`; a `termSeconds` chosen from the permitted terms; a `fillDeadline`, which is the launch window beyond which nothing can fill it; and a `salt`. A signature is only needed when a third party submits the request, so a borrower who calls `originate` personally does not need one. Calling `cancelRequest` retracts a signed request by consuming its digest, the same way a fill does.

## Supported signature types

Verification goes through `SignatureChecker` from OpenZeppelin, which accepts either an ECDSA signature produced by an EOA or EIP-1271 validation from a smart contract wallet. Contract wallet support is not optional here, because Robinhood Chain supports account abstraction through both ERC-4337 and EIP-7702 and a large share of wallets are therefore contracts.

## Cancellation, expiry and partial fills

Calling `cancel(nonce)` flips a single bit in the maker's bitmap, which invalidates all offers that use that nonce. `cancelWord(wordPos, mask)` flips several bits at once within the block of 256 nonces beginning at `wordPos * 256`. Each takes effect at once, can safely be repeated, and emits `OfferCancelled` or `OfferWordCancelled` respectively.

The hub tracks principal drawn against every offer in `filled[digest]`, and `remainingCapacity(offer)` reports how much is still available. For example, a standing offer whose `principalMax` is 50,000 USDG and which funds 10,000 today keeps 40,000 to lend until it expires or is cancelled. Every fill emits `OfferFilled` carrying the cumulative amount, and if a fill was booked during a rollover that fails to clear, `OfferReleased` returns it. An offer simply stops verifying after `expiry`, with no transaction required.

## The relayer's part

Unfilled offers are held by relayers, which are stateless services speaking REST and WebSocket. A relayer keeps the signed messages from both sides, validates them against the chain for balance, allowance, eligibility, nonce and expiry, responds to queries narrowed by any of token, tier, term, APR and LTV, and suggests a fill for any request. It holds no privileged power. Anybody may operate one, every relayer's book contains the same signed messages, and the indexer keeps a copy. More detail is in [Services off the chain](../architecture/services.md).

The matching rule is deterministic. From the lend side, the relayer keeps only offers that share the request's `loanToken` and `termSeconds`, reference its `collateralToken` either directly or via the tier sentinel, have an `aprBps` no higher than the borrower's cap, have a `maxLtvBps` no lower than the request LTV, and remain live (not expired, not cancelled, with capacity remaining). It ranks those by APR from cheapest upward and uses them in turn until the principal is reached. Before signing, the borrower can review the syndicate, its blended APR and the terms of every slice. Sequence also matters on chain, since the hub fills offers in the order they are passed in.

## What originate checks, step by step

Calling `originate(request, borrowerSignature, offers, signatures)` gives back the id of the new loan. If any step fails, the entire call reverts.

1. The pause on opening loans must be off (`NewLoansPaused`).
2. The current time must be no later than `fillDeadline`, which marks the end of the launch window (`RequestExpired`).
3. Both `principal` and `collateralAmount` have to be above zero (`ZeroAmount`).
4. The request digest must not have been used before, and it is flagged as used at this point (`RequestAlreadyUsed`).
5. If someone other than the borrower is calling, they must hold `RELAYER` and supply a valid `borrowerSignature` (`BadRequestSignature`).
6. The borrower has to hold `BORROWER`.
7. Next come the risk config checks that do not depend on price. The term must be permitted (`TermNotAllowed`), both tokens must be enabled (`TokenNotEnabled`), the loan token must equal the quote token of the collateral's feed (`LoanTokenMismatch`), and the exposure cap must have headroom (`ExposureCapExceeded`).
8. The guard updates the price and stores a move-cap checkpoint. The request fails if the feed is paused, the valuation comes out as zero, the feed is stale during an open session (`OracleUnusable`), or the sequencer restarted recently enough to be inside its grace period (`SequencerGrace`). When the session is closed, or the quote is stale while closed, the tier ceiling drops by the closed-market haircut. The request LTV, defined as principal divided by collateral value, has to stay at or below the ceiling (`LtvTooHigh`).
9. The hub writes the loan with a fresh id, status `Active`, a start time of now, and maturity equal to start plus term.
10. There must be as many signatures as offers (`LengthMismatch`). Each offer is then handled in sequence. It is authenticated (`OfferExpired`, `OfferWasCancelled`, `BadOfferSignature`). Its side is checked (`OfferSideNotSupported`), and so is its fit with the request on tokens, term, rate, LTV limit and `requestId` (`OfferMismatch`). It is rejected if the request is already fully covered (`OverFilled`) or the offer has nothing left (`OfferExhausted`). The fill size is whichever is smaller, remaining capacity or the remaining shortfall, and must respect `principalMin` (`FillTooSmall`). The maker must hold a lender role. The fill is recorded (`OfferCapacityExceeded`). Principal comes from the vault when an adapter is connected and the offer sets bit 2, and from the maker's allowance otherwise. Finally the slice is minted, subject to the limit of 32 per loan (`TooManySlices`), with its minimum interest locked in.
11. Total fills have to equal the requested principal precisely (`UnderFilled`).
12. Exposure is increased and the collateral is transferred into escrow. Because this happens only after the fills succeed, a request that cannot be covered never moves anything out of the borrower's wallet.
13. The origination fee, equal to `originationBps` of principal, is sent to the `FeeVault`, and the remainder goes to the borrower.
14. The hub emits `LoanOriginated`, carrying the loan id and borrower, both tokens, the collateral amount, the gross principal and the maturity. Each slice has already been announced through its own `SliceCreated`.

Checking a batch of signatures is the most expensive part of this call. A Stylus verifier in Rust is planned on the roadmap to bring that cost down; see [The contract set](../architecture/contracts.md) for where things stand today.

## Example: filling a request

Suppose a borrower wants to borrow 20,000 USDG over 30 days, putting up 250 NVDA Stock Tokens and accepting at most 9.00% APR. With NVDA at 176.40, the collateral is worth 44,100 USDG, which gives a request LTV of 45.4%, comfortably under the 55% ceiling for Tier A. These three standing offers qualify:

| Lender        | Rate (APR) | Available   | Max LTV |
| ------------- | ---------- | ----------- | ------- |
| 0x71c3...9e4a | 8.50%      | 5,000 USDG  | 50%     |
| 0xb02f...17d3 | 8.90%      | 10,000 USDG | 55%     |
| 0x4e88...c0b1 | 9.00%      | 5,000 USDG  | 50%     |

The principal-weighted blended APR comes to 8.825%. A single `originate` creates three slice tokens and sends the borrower 19,950 USDG, which is 20,000 minus the 0.25% origination fee. Tier A liquidates at an LTV of 70%, so the health factor is 44,100 × 0.70 = 30,870 divided by 20,000, giving 1.54.
