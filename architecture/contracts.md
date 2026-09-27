---
description: A reference to the Spatial contracts deployed on Robinhood Chain, covering what each one may do and the calls, events, flags and roles that integrations rely on.
---

# Contract reference

Spatial keeps all of its on-chain code in one Foundry project, the repository's `contracts/` folder, and targets Robinhood Chain (chain ID 4663). What follows documents how the deployed contracts behave. For compiling, running the tests and finding live addresses, go to [Build and deploy](build-and-deploy.md).

{% hint style="info" %}
Compiler settings: Solidity 0.8.26 for the Cancun EVM, the IR pipeline, 100 optimizer runs, and OpenZeppelin 5.2 as the library base. Source for every deployment is verified on Blockscout. [Assurance](assurance.md) covers testing, audits and the safeguards kept on in production.
{% endhint %}

## What ships

| Contract | Responsibility | Who can alter it |
| --- | --- | --- |
| `CreditHub` | Checks offers, holds collateral in escrow, sends out principal, records every loan and slice, applies repayments and exposes the hooks both auctions drive | No one, beyond a single `wire()` call |
| `QuoteBook` | The hub's base contract. It owns an EIP-712 domain named `Spatial` at version `1`, keeps a nonce bitmap per maker and a running fill total per offer digest, and checks signatures as ECDSA or EIP-1271 | No one |
| `LiquidationAuction` | Sells collateral at a falling price; also handles restarts, payment in kind, and lenders who opt out while the market is shut | No one |
| `RolloverMarket` | Runs the rollover auction at a rising rate and keeps lender acceptances until the auction either clears, fails or is called off | No one |
| `OracleGuard` | Reads a Chainlink feed for each token, plus an optional Data Streams adapter, and checks staleness, the session, sequencer state, pauses and the move cap | Governance sets feed configuration |
| `PermissionRegistry` | Passes every role query through to whichever adapter `RiskConfig` points at | Adapter changes go through the timelock |
| `CredentialRegistry` | The bundled adapter, an EAS-style store that approved issuers write to | Issuer list changes go through the timelock |
| `SliceNote` | ERC-721 collection "Spatial Slice Note" (`SPSN`); one token per slice, movable only to a wallet that qualifies as a lender | Governance sets the base URI |
| `ReserveAdapter` | Holds idle lender USDG in a single whitelisted ERC-4626 vault and releases it at the moment it is needed | Each adapter is bound to one vault at deployment |
| `RiskConfig` | Every parameter, the timelock, the guardian's pause switch and the bootstrap flag | The owner while bootstrapping; the timelock once `finishBootstrap()` has run |
| `FeeVault` | Receives protocol fees | Governance makes withdrawals |

Beneath them are three libraries. `LoanTypes` defines the structs, enums and flag bits, and also carries the `Permissions` library of role ids. `QuoteHash` does the EIP-712 hashing and derives the rollover key. `InterestMath` accrues simple interest per second on principal only; nothing compounds.

## Principles behind the design

* **Immutable, with no operator.** `CreditHub`, `QuoteBook`, `SliceNote`, `LiquidationAuction` and `RolloverMarket` cannot be upgraded and give nobody special powers. A new version means a fresh deployment, and loans opened under an older deployment run to completion on it.
* **Code and settings live apart.** Everything governance is able to adjust sits in `RiskConfig`, gated by a timelock.
* **One market for each token pair.** For every collateral token the tier, the exposure cap and the feed settings are looked up separately, and a loan must be denominated in the token its feed quotes.
* **Shaped for Arbitrum Nitro.** Every clock reading is `block.timestamp`, offer calldata stays small, and each public entry point remains callable through the L1 delayed inbox.

## Messages, flags and states

The two signed message types are declared in `LoanTypes` and hashed by `QuoteHash`. Each type string lists fields in exactly the struct's order, and `Side` is hashed as a `uint8`.

```solidity
struct LendOffer {
    address maker;
    Side    side;            // the hub only fills Lend
    address collateralToken; // a specific token, or address(uint160(tier)) to accept any token in that tier
    address loanToken;
    uint256 principalMin;    // minimum fill, waived when the fill uses up the offer
    uint256 principalMax;
    uint16  aprBps;
    uint16  maxLtvBps;       // highest loan-to-value the lender agrees to fund
    uint32  termSeconds;
    uint40  expiry;
    uint256 nonce;           // used to cancel; filling does not consume it
    bytes32 salt;
    bytes32 requestId;       // zero = open to any request; a request digest = that request only; rolloverKey(loanId) = that rollover only
    uint8   flags;
}

struct BorrowRequest {
    address borrower;
    address collateralToken;
    address loanToken;
    uint256 collateralAmount;
    uint256 principal;
    uint16  maxAprBps;
    uint32  termSeconds;
    uint40  fillDeadline;    // the Launch window
    bytes32 salt;
}
```

A lender sets flag bits on the offer, and the slice created from it inherits them:

| Constant | Bit | What it does |
| --- | --- | --- |
| `FLAG_SELF_LIQUIDATE` | 1 | When the loan is liquidated, this lender takes collateral priced by the guard instead of a cut of what the sale raises, limited to the slice's principal-weighted portion of the escrow |
| `FLAG_NO_CLOSED_MARKET_LIQUIDATION` | 2 | Keeps the slice out of any auction begun while equity trading is closed |
| `FLAG_PARK_IDLE` | 4 | Fills are drawn from, and repayments returned to, the vault that `ReserveAdapter` fronts |

Apart from `None`, which marks an id never assigned, `LoanStatus` has six states: `Active`, `Refinancing` and `Liquidating` while a loan is open, `Repaid` and `Settled` once it has closed, and `Defaulted` after a rollover fails. A loan stays `Active` for its whole term, during the post-maturity grace window, and beyond that right up until an auction opens. `Session` takes one of three values: `Regular`, `Extended` or `Closed`.

## Role ids
Each role id is `keccak256("spatial.role.<NAME>")`. The names are `BORROWER` and `RELAYER` on the borrowing side, `LENDER_PROFESSIONAL` and `LENDER_RETAIL` for lenders, and `LIQUIDATOR` for auction buyers. `PermissionRegistry.isLender` returns true when a wallet holds either of the two lender roles.

## CreditHub

### Calls open to borrowers and lenders

| Function | Effect |
| --- | --- |
| `originate(request, borrowerSig, offers[], sigs[])` | Fills a request from the supplied offers, in the order given, inside one atomic transaction. If the borrower sends it, the signature argument is not checked; any other sender must hold `RELAYER` and supply a valid borrower signature. The offers have to add up to exactly `request.principal`. |
| `repay(loanId, amount)` | Anyone may call it and no pause can block it. Works in `Active` and `Defaulted`. Interest is cleared before principal, and each part is shared among slices in proportion to what each slice is owed. Paying off the whole debt burns the slices and hands back the escrow. |
| `addCollateral(loanId, amount)` | Any address can add collateral, in every state where repayment is allowed and also while a rollover is running. |
| `cancelRequest(request)` | Callable by the borrower alone. Marks the request digest as used, so a relayer still holding the signature has nothing it can fill. |
| `cancel(nonce)`, `cancelWord(wordPos, mask)` | Inherited from `QuoteBook`. The first retires a single nonce; the second retires every nonce selected by a mask within one 256-nonce word. |

### Views

`getLoan` and `getSlice` return the stored records. `debtOf` gives principal and interest up to the current second, with each slice held to at least its minimum interest. `healthFactor` is in WAD and already deducts the closed-market haircut whenever the quote is closed or stale. `ltvBps` divides debt by collateral value, and `exposure(token)` reports principal outstanding per collateral token. `QuoteBook` adds `requestHash`, `offerHash`, `domainSeparator` and `remainingCapacity(offer)`. Two reads that one might look for on the hub are found on other contracts: `blendedAprBps(loanId)` sits on `RolloverMarket`, and `isPastGrace(loanId)` sits on `LiquidationAuction`.

### The order of checks inside `originate()`

1. New loans are not paused, `fillDeadline` has not passed, and neither principal nor collateral is zero.
2. The request digest has never been used. It is marked used at this point, which rules out replay.
3. When someone other than the borrower is calling, that caller must be a `RELAYER`, and the EIP-712 signature supplied for the borrower has to verify.
4. The borrower's wallet holds the `BORROWER` role.
5. Checks that need no price: the term is on the allowed list; the loan token is enabled and matches `OracleGuard.quoteToken(collateral)`; the collateral token is switched on too; and the new principal fits under the token's exposure cap.
6. `OracleGuard.refresh()` is called, which stores a move-cap checkpoint. The call reverts if the market is paused, if the collateral values at zero, if the price is stale during an open session, or if the sequencer is still in its grace period. A stale price while the market is closed goes through, with the haircut applied as for any closed-session quote.
7. The request's LTV must not exceed the tier's `maxLtvBps`, reduced by `closedHaircutBps` if the haircut is in force.
8. Each offer is examined in turn. It must be unexpired, carry a live nonce and a signature from its maker, and sit on the lend side. For collateral it names either the same token as the request or the tier wildcard. Its loan token, term, rate (no higher than the borrower's cap), LTV limit and `requestId` must suit the request. It needs spare capacity, and the fill must reach `principalMin` or else drain the offer. Finally the maker has to pass the lender check, and the slice count must stay within `maxSlicesPerLoan`.
9. Funds move. Each lender's principal comes from their wallet, or through `ReserveAdapter` when the offer sets `FLAG_PARK_IDLE`, and every offer produces one slice. Collateral is pulled into escrow, the origination fee goes to `FeeVault`, and the borrower receives principal net of that fee.

### Hooks reserved for the two auctions

The liquidation auction relies on `transferCollateral`, `beginLiquidation`, `settleInKind`, `splitForLiquidation` and `finalizeLiquidation`. `consumeOffer`, `releaseOffer`, `pullFromIdle`, `beginRefinance`, `cancelRefinance`, `clearRefinance` and `markDefaulted` are what the rollover market uses. Each of these twelve carries the `onlyAuctions` modifier, which lets through only the two addresses stored by `wire()`. Either of those addresses can reach any hook, and the status check inside each hook rejects calls that do not suit the loan's current state. One further hook, `onSliceTransfer`, answers only to `SliceNote` (anything else reverts with `NotSliceNote`) and points a slice's payouts at its new holder when the token is transferred.

The owner of `RiskConfig` calls `wire(liquidationAuction_, rolloverMarket_, reserve_)` exactly once. Passing a zero reserve turns idle parking off; a non-zero reserve has to be backed by a whitelisted vault. Its collaborators can be read back through `riskConfig()`, `permissions()`, `oracleGuard()`, `sliceNote()`, `feeVault()`, `liquidationAuction()`, `rolloverMarket()` and `reserve()`. The constructor fixes the first five of these, and `wire()` sets the remaining three.

## LiquidationAuction

| Function | What it does |
| --- | --- |
| `startAuction(loanId)` | Open to all, and whoever calls it is recorded as the keeper. A loan qualifies if its health factor is below 1.0, if it is `Defaulted`, or if it is `Active` with maturity plus the grace window behind it. The call is refused while liquidations are paused, while the guard reports the market paused (`GuardPaused`), during sequencer grace, or when the price is stale in an open session. |
| `restart(loanId)` | Anyone may call it once the window has run out without reaching the target. It rebuilds the schedule from a new quote; proceeds already raised still count, and the keeper share stays with the original keeper. |
| `currentPrice(loanId)` | Begins at the quote multiplied by (1 + start premium), falls linearly to the quote multiplied by the floor ratio, and stays there. In a closed session the ratio is `floorBpsClosed`; otherwise it is `floorBpsRegular`. |
| `buy(loanId, collateralAmount, report)` | Requires `LIQUIDATOR` and stops while liquidations are paused. The quantity is capped both by the escrow balance and by the shortfall against the target. Tokens with a stream adapter need a report, which is verified against the feed. Payment goes directly to the hub, and the auction settles when proceeds hit the target or the escrow runs dry. `buyWithLimit` does the same with a maximum acceptable price. |

Opening an auction runs these steps in order:

1. Take a fresh price with `refresh`.
2. Call `beginLiquidation`. If a rollover is in progress, it is abandoned here.
3. In a closed session, gather the slices flagged `FLAG_NO_CLOSED_MARKET_LIQUIDATION` and use `splitForLiquidation` to move them, with collateral in proportion, into a separate `Active` loan. If no slice is left behind, the call reverts.
4. Settle every slice carrying `FLAG_SELF_LIQUIDATE` by handing over collateral valued at the checkpointed price, never above its pro rata share.
5. If no collateral or no slice remains, finalise immediately with zero proceeds.

`finalizeLiquidation` computes the penalty as `penaltyBps` of the debt and distributes proceeds in a set sequence, each step taking only what remains: the keeper's part of the penalty; lender claims, split according to how much each is owed; the lenders' part of the penalty; the protocol's part together with the interest share on interest recovered; and finally leftover loan token and any unsold collateral to the borrower. The loan ends `Settled` even when lenders take a loss.

## RolloverMarket

On chain the call is named `openRefinance`; these docs refer to the process as a rollover auction.

| Function | What it does |
| --- | --- |
| `openRefinance(loanId)` | Borrower only, for an `Active` loan that has not reached maturity. The rate starts at the blended APR of the current slices and rises in a straight line over `duration` until it reaches that figure plus `capSpreadBps`. |
| `accept(loanId, offer, sig)` | Any address may submit a lender's signed offer for them. The offer's APR has to be at or below the current rate, and the resulting slice is booked at that current rate. Tokens and term must match the loan, `requestId` must be zero or `rolloverKey(loanId)`, and the offer's LTV limit must cover the loan's present LTV. The money is held by the market, and as soon as acceptances reach debt plus the rollover fee, clearing happens in that same call. |
| `fail(loanId)` | Open to anyone once the window closes without the target being met. Each acceptor is refunded, the offers get their capacity back, and the loan moves to `Defaulted`. |
| `cancelRefinance(loanId)` | Borrower only, any time before clearing. All acceptances are returned and the loan goes back to `Active` on its existing terms. |
| `blendedAprBps`, `currentRate`, `target`, `acceptances` | Views for the starting rate, the rate right now, debt plus fee, and the funded acceptances in the order they arrived. |

Clearing runs through `clearRefinance`, which pays every outgoing slice its principal plus interest less the interest share, forwards the rollover fee to `FeeVault`, and restarts the loan for the new syndicate: a new start time, a term of the same length, and principal equal to the old debt plus the fee. If the health factor then sits under 1.0, the whole call reverts. `cancelForLiquidation` can only be called by the hub (others get `NotHub`), and it unwinds a rollover when a liquidation auction starts on that loan.

## OracleGuard

`quote(token)` is the read-only path, and it does not revert because a price is stale or a market is paused. It returns `price` (loan-token base units per whole collateral token) along with `updatedAt`, `session`, `paused`, `stale`, `multiplier` and `sequencerGrace`. The `multiplier` field is the ERC-8056 `uiMultiplier`, shown for reference only and never applied, because Chainlink equity feeds include it already.

`refresh(token)` is the path that writes state, and it runs at origination and whenever an auction opens or restarts. A move larger than `moveCapBps`, measured against a checkpoint taken within the last `moveCapWindow`, pauses the market. That pause stays until governance calls `resume(token)`, and governance may also call `pause(token)` itself.

A token with a market-status source takes its session from there, using the Chainlink Data Streams status codes: 1 means regular, 2 extended, 5 closed. Tokens without such a source follow a weekday timetable in UTC. Out of the box it puts regular trading between 14:30 and 21:00 and extended trading between 09:00 and 01:00 the next day. Every session carries its own staleness limit. `verifyStreamReport(token, report)` hands the report to the adapter configured for that token, then refuses it if it is older than the session limit or strays from the feed by more than `streamDivergenceBps`. Tokens are set up with `configureFeed`, and the uptime feed is set with `setSequencerFeed`.

## RiskConfig

While bootstrapping, the owner can call setters directly. Calling `finishBootstrap()` ends this permanently. From then on a setter can only run as part of a batch. The batch is queued by `schedule(calls, salt, rationale)` and run by `execute(calls, salt)`, which works only after `delay` has elapsed and before the fixed 14-day `GRACE_PERIOD` lapses. A queued batch is withdrawn with `cancel(id)`. Once bootstrap is over, the delay has a floor of one hour. Handing over ownership takes two calls, `transferOwnership` followed by `acceptOwnership`.

`setPaused(newLoans, liquidations)` does not go through the timelock. The owner and the timelock can move either flag in either direction, whereas the guardian can only switch a flag on and never off.

Each setter finishes by logging `ParamChanged(key, subject, value)`. Other contracts reach the settings through `riskConfig()`, and they cover:

* per tier: the opening LTV ceiling, the liquidation LTV and the closed-market haircut;
* per collateral token: its tier and its exposure cap;
* the list of enabled loan tokens and the list of permitted terms;
* `LoanParams`: minimum interest period, grace window, sequencer grace and the slice limit per loan;
* `AuctionParams`: start premium, the regular and closed floors, duration, penalty, keeper share and lender share;
* `RefinanceParams`: duration and cap spread;
* `FeeParams`: origination, interest share, rollover fee and idle yield share;
* the whitelist of vaults, the approved credential issuers and the eligibility adapter;
* the guardian address and the timelock delay.

## The supporting contracts

* **`CredentialRegistry`** is the adapter currently in use. Approved issuers may call `attest` and `revoke`, and any one of them may revoke any record. A record counts only while it is unrevoked, unexpired and written by an issuer that is still approved, which means removing an issuer cancels everything it issued.
* **`SliceNote`** mints and burns only when the hub asks. On a transfer, the recipient has to pass `requireLender`, and the hub is told before ownership changes hands. Governance can set `baseURI`. The hub address is fixed once through `setDesk` and is readable as `hub()`.
* **`ReserveAdapter`** records each lender's vault shares. Lenders call `deposit`, `withdraw` and `withdrawAll`, while the hub calls `withdrawFor` and `depositFor`; a withdrawal larger than the lender's position reverts with `InsufficientReserve`. Deposits re-check the whitelist each time, so removing a vault stops fresh money going in without locking anyone in, and if a repayment cannot go into the vault it is paid to the lender's wallet instead. Like `SliceNote`, it learns the hub address through a one-time `setDesk`.
* **`FeeVault`** accepts fees as ordinary transfers, and only the owner of `RiskConfig` can `withdraw`.

## Emitted events
| Contract | Events emitted |
| --- | --- |
| `QuoteBook` | `OfferCancelled`, `OfferWordCancelled`, `OfferFilled`, `OfferReleased` |
| `CreditHub` | `CollateralAdded`, `InKindLiquidation`, `LiquidationFinalized`, `LiquidationStarted`, `LoanDefaulted`, `LoanOriginated`, `LoanPartiallyRepaid`, `LoanRepaid`, `LoanSplit`, `RefinanceCancelled`, `RefinanceCleared`, `RefinanceStarted`, `RequestCancelled`, `SliceCreated`, `SliceTransferred`, `Wired` |
| `LiquidationAuction` | `AuctionStarted`, `AuctionBuy`, `AuctionRestarted`, `AuctionSettled` |
| `RolloverMarket` | `RefinanceOpened`, `RefinanceAccepted`, `RefinanceClearedEvent`, `RefinanceFailed`, `RefinanceCancelledEvent` |
| `OracleGuard` | `Checkpointed`, `FeedConfigured`, `MarketPausedEvent`, `MarketResumed`, `ScheduleSet`, `SequencerFeedSet` |
| `CredentialRegistry` | `Attested`, `Revoked` |
| `RiskConfig` | `BootstrapFinished`, `DelayChanged`, `EmergencyPause`, `GuardianChanged`, `OperationCancelled`, `OperationExecuted`, `OperationScheduled`, `OwnershipTransferCancelled`, `OwnershipTransferStarted`, `OwnershipTransferred`, `ParamChanged` |
| `SliceNote` | `DeskSet`, `BaseURISet`, `BatchMetadataUpdate` |
| `ReserveAdapter` | `Deposited`, `Withdrawn`, `DeskSet`, `ParkingSkipped` |
| `FeeVault` | `Withdrawn` |

## Guarantees across the set

* A loan only exists if all of its offers passed verification, all lender principal arrived and the collateral reached escrow. Short of that, the entire call reverts.
* Borrowers can repay during `Active` (covering the term, the post-maturity grace window and any time after) and during `Defaulted`. It only stops once a liquidation auction has opened.
* If one slice ends up short, the loss stays with that slice. Nothing is spread across other slices or other loans.
* Collateral is priced only by the feed set up for the precise token sitting in escrow. Wrapped tokens and derived rates are not accepted.
* Auction proceeds always follow the same waterfall: keeper share, then lender claims, then the lenders' penalty share, then the protocol's penalty share, then any surplus back to the borrower.
* Governance keys have no route to escrowed collateral, to lender principal or to vault balances.

## Staying under the size limit
The biggest contract is `CreditHub`, at 24,451 bytes of runtime bytecode. That leaves 125 bytes of headroom under the EIP-170 ceiling of 24,576 bytes, a limit Arbitrum Nitro enforces, and it is why the optimizer runs only 100 times. There are plans to port the offer verifier to Stylus, which would create more room.
