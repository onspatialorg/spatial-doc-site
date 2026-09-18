---
description: OracleGuard is the single contract that turns what Chainlink reports into the price that the hub and the auction act upon, once it has identified the trading session and tested the value for staleness, a pause or an abnormal move.
---

# Prices, sessions and oracle checks

The hub never queries Chainlink itself, and neither does the liquidation auction. Each of them goes through `OracleGuard`, which reads the feed, works out the session, applies its checks and hands back a quote with flags attached. A quote does not revert because the market is stale or paused. The flags carry that information and the calling contract chooses what to do.

## Contents of a quote

Calling `quote(collateralToken)` gives back the price expressed in loan-token units, plus the session, `updatedAt`, `paused`, `stale`, `sequencerGrace` and the multiplier from ERC-8056. `refresh` hands back an identical quote and also records a move-cap checkpoint. For any token, `feedConfig` shows its configuration, `checkpoint` shows the latest checkpoint and the pause flag, and `hasStream` reports if a stream adapter for Data Streams has been attached. Both the Explorer and Telemetry display these values live, token by token.

## The inputs

| Source | How it arrives | What relies on it |
| --- | --- | --- |
| Chainlink **Data Feeds**, running 24/5 | Pushed on chain whenever the deviation threshold or heartbeat is hit. | The LTV at origination, the health factor and warnings. |
| **Data Streams** from Chainlink, RWA schema v11 | Pulled. Whoever buys in the auction supplies a signed report, and the stream adapter checks it on chain via the VerifierProxy. `marketStatus` is part of the report. | The settlement price of a liquidation auction. |
| Chainlink **Sequencer Uptime Feed** | Pushed. | The grace window after an outage, explained on the [sequencer page](sequencer.md). |
| ERC-8056 `uiMultiplier()` | Taken straight from the Stock Token's own contract. | Accounting in share terms and what the Explorer shows. The multiplier is already built into Chainlink's answer, so `OracleGuard` passes it through for reference and does not apply it. |

Having both Feeds and Streams gives a cross-check at no extra cost. If a stream report differs from the feed by more than the tolerance (set at 2% in the deployment), it is refused and the purchase reverts.

## Regular, extended and closed sessions

On a stock exchange, regular hours total roughly 32 a week, slightly more once extended hours are counted, and the rest of the time trading is shut. Every quote from `OracleGuard` is tagged **regular**, **extended** or **closed**. When a market-status source has been set up, the tag comes from Chainlink's status codes: 5 means closed, 2 means extended, and every other code means regular. When no such source exists, a fixed timetable is used instead: Saturday and Sunday are closed, and on weekdays the regular window (UTC) runs 14:30 to 21:00 and the extended window opens at 09:00 and closes at 01:00 on the next day. A timetable has no knowledge of exchange holidays, which is why every production token is given a status source.

Staleness is bounded separately for each session. The deployed values:

| Session | Maximum age of a price | Effect |
| --- | --- | --- |
| Regular | 1 hour | The tier LTVs are used as they stand. |
| Extended | 2 hours | The tier LTVs are used as they stand. |
| Closed | 4 days | Both the liquidation LTV and the max LTV are cut by the closed-market haircut set for the tier. |

Monday's opening price can land far from Friday's closing price, and that is what the haircut is for. Because the liquidation LTV is lowered as well, a position that seems comfortable on a Friday afternoon has to carry more headroom over the weekend. Borrowers close to the line are warned by the platform before trading stops rather than once it has restarted.

## Each check and what it stops

| Trigger | Result |
| --- | --- |
| A zero or negative answer, a zero or future timestamp, or a price that becomes zero after rescaling | `quote` reverts. |
| During regular or extended hours, `updatedAt` exceeds the staleness bound | Origination reverts, and auctions cannot be opened. The health factor uses the closed-market haircut. |
| With the market shut, `updatedAt` exceeds the staleness bound | This is normal. Origination goes ahead with the haircut applied, the health factor uses it too, and the floor of the auction follows the closed-session ratio. |
| A `refresh` sees the price move further than the 25% cap from a checkpoint recorded in the previous hour | The market pauses. No origination and no new auctions until governance reviews it and calls `resume`. Repaying does not depend on a price, so it carries on. |
| The token returns true from `oraclePaused()`, or governance has invoked `pause` | For as long as the flag stands, the market is paused. When the token raises it, a corporate action is usually in progress. Origination and liquidation halt, and repayment continues. |
| A stream report falls outside tolerance against the feed, or its timestamp exceeds the staleness bound for the session | The purchase in the auction reverts. |

## Pricing the escrowed token and nothing else

Each market's configuration points to the feed for exactly the token sitting in escrow. No wrapper, vault share or exchange rate between tokens is ever used as a price source. This rule is a response to the wGOOGLx loss at Edel Finance in July 2026, where the oracle for the underlying was accurate but the rate on a wrapper had been inflated 78 times and treated as collateral value. The [precedents page](precedents.md) tells the full story.

## Updating a feed

For a given collateral token, `configureFeed` sets the feed, the loan token, a staleness bound per session (three in all), the move cap plus the window it covers, the tolerance for stream reports, and, if wanted, sources for market status, pausing, the multiplier and a stream. Only the owner of `RiskConfig` can call it, and every call emits `FeedConfigured`. As with other risk parameters, any change to a feed appears in the governance log alongside the reasoning for it.
