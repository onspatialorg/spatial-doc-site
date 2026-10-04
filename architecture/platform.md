---
description: The engineering behind onspatial.org/platform, covering its six routes, how Privy sessions reach Supabase, the database schema, EIP-712 signing, settlement on-chain and the visual design system.
---

# Inside the platform

The platform hardcodes none of its data. Markets, prices, parameters, requests, offers, loans and events are all fetched from Supabase at render time, so the screen reflects the live book and never a frozen copy. It serves as Spatial's reference front-end and its relayer at once, in one app hosted at onspatial.org/platform. The stack is Next.js 16 using the App Router, React 19, TypeScript and Tailwind CSS 4, with Privy handling sign-in, Supabase (Postgres) storing the loan registry alongside the common book of orders, and viem handling typed-data hashes and settlement. Pages are bright and calm, and each one is arranged around the figures it reports.

## Routes

| Path | Who can open it | Contents |
| --- | --- | --- |
| `/platform` | Anyone | Telemetry. Principal outstanding, open capacity against demand, and liquidations; a markets table with each tier's limits, its price and an indicative rate updated live; the requests currently open |
| `/platform/borrow` | Signed-in users | A form for a new request (collateral, principal, maximum rate, term, and a launch window measured in hours) that works out LTV, health factor and the origination fee while you type; your requests, each showing how much the book now covers; the settle action |
| `/platform/lend` | Anyone, with extra features after sign-in | Open requests; a form for posting a standing or a targeted offer, with its flags; your own offers with the capacity still available on each, and a way to cancel them |
| `/platform/positions` | Signed-in users | Loans you hold as borrower (debt, interest accrued so far, health factor, with actions to repay or top up) and slices you hold as lender (principal, APR, accrued interest, health, maturity date) |
| `/platform/explorer` | Anyone | The Explorer. It lists the loan registry, every loan's syndicate and flight log, how concentrated each token is relative to its exposure cap, and the parameters in force |
| `/platform/settings` | Signed-in users | Your profile, plus each role's attestation issued to your wallet |

A handful of product names come up again and again. Telemetry means the protocol's live figures. The Explorer is the public view of all loans, open to anybody. A flight log is the event history of a single loan, with a link from every entry to its transaction on Blockscout. The launch window is the deadline by which a borrow request must fill. For a tour, read [Reading the Explorer](../guides/explorer.md).

## Signing in

Users authenticate through Privy with email, a Google account or any EVM wallet. Anyone without a wallet receives an embedded one on Robinhood Chain, and that wallet then signs all of their requests and offers.

## Connecting Privy and Supabase

Privy and Supabase know nothing about each other, so a server route links them:

1. Public data (markets and their prices, parameters, the book, loans, slices, events) is fetched straight from the browser, authenticated by nothing more than the anon key.
2. Every action taken for a specific user goes to `/api/platform/db` as a `POST` with the Privy access token in its `Authorization` header. That covers loading a profile or the list of attestations, posting a request or offer, cancelling, settling, repaying and topping up.
3. The route verifies the token using `@privy-io/server-auth`. A missing or bad token gets a 401, and an operation it does not recognise gets a 400. Arguments are validated by `src/lib/platform/ops.ts`, and the operation then runs on a Supabase client that holds the service-role key and sends the verified user id in an `x-spatial-actor` header. Inside the schema, `app_user_id()` trusts that header only for callers whose JWT carries the `service_role` role, so the ownership checks in each `security definer` function are made against the real user.

The service-role key never reaches the browser, and if someone supplies an anon or publishable key in its place, the route will not start. Anonymous reads remain limited by row-level security, and every write still goes through database functions that enforce the relevant rules.

## Schema

`supabase/schema.sql` defines one table for every structure that exists on-chain:

| Table | Mirrors on-chain | Written by |
| --- | --- | --- |
| `profiles` | No on-chain counterpart | Each user, limited to their own row |
| `markets` | Per-token and per-tier settings held in `RiskConfig` | The service role |
| `oracle_prices` | `OracleGuard.quote()`. Rows are only ever appended; the latest per market is available through the `market_latest_prices` view | The service role, fed by a keeper or a scheduled job |
| `protocol_params` | Parameters from `RiskConfig`, along with the contract addresses stored under `deployment` | The service role |
| `attestations` | `CredentialRegistry` | The service role, acting as KYC issuer |
| `borrow_requests` | The request half of the relayer's book | Borrowers, through the API route |
| `offers` | The offer half of the relayer's book, with each row holding a signed EIP-712 message | Lenders, through the API route |
| `loans`, `slices`, `loan_events` | State held by `CreditHub` | Only database functions |

`protocol_params` has eight keys: `deployment`, `chain`, `tiers`, `terms`, `loan`, `fees`, `auction` and `refinance`. Telemetry figures come from the `protocol_stats` view, and `has_role(wallet, role)` checks eligibility against the mirrored attestations.

All state transitions are `security definer` functions. `settle_request` turns a covered request into a loan, `record_repayment` and `record_collateral_added` log repayments and top-ups, and `cancel_offer` and `cancel_request` withdraw entries from the book, while `loan_debt()` works out the amount owed. Because each transition lives in the database, it can enforce exactly the checks that `CreditHub` enforces on-chain: expiry, term, the APR ceiling, scope by tier or by token, capacity remaining, the minimum fill, and complete coverage.

## Typed-data signing

Requests and offers are signed as EIP-712 typed data whose primary types are `BorrowRequest` and `LendOffer`. The struct definitions and the domain (`CreditHub` as verifying contract, the chain id, version `1` and the name `Spatial`) are identical to those in `contracts/src/libraries/QuoteHash.sol`. That lets the hub, or any relayer, check a stored message afterwards, which is why the typed-data hash is kept beside each signature. When a standing offer covers an entire tier, its collateral token field holds the tier marker `address(uint160(tier))`. Flags are set as bits: `SELF_LIQUIDATE` is 1, `NO_CLOSED_MARKET_LIQUIDATION` is 2 and `PARK_IDLE` is 4. Signatures come from Privy's `signTypedData` and use no gas.

## Settlement

Contract addresses live in the `protocol_params` row whose key is `deployment`, under `chainId`, `CreditHub`, `LiquidationAuction`, `RolloverMarket`, `OracleGuard`, `PermissionRegistry` and `SliceNote`, copied from `contracts/deployments/4663.json`. Once that row has a `CreditHub` address, settling a request sends a real `originate` transaction through viem's `writeContract`, signed with the user's Privy wallet, and the transaction hash is stored on the loan row. If the address is absent, the match is recorded in the book alone and the panel makes that clear. Offers are matched greedily on APR, lowest first, following the rules the relayer documentation sets out. [Off-chain services](services.md) and [Offers, requests and origination](../mechanics/offers-and-origination.md) give the detail.

## Design system

Website and app are drawn from the same design system, so they come across as one product.

**Type.** Two families. Bricolage Grotesque, set at 600 with tight tracking, is the display face, used for the wordmark, page headings and big figures. Geist handles everything else: smaller headings, labels, buttons, tables and body copy. Figures of every kind (amounts, addresses, hashes, countdowns) use Geist with tabular numerals, since those values are the reason each screen exists.

**Colour.** There is a single accent, Lagoon (`#0a8782`), with a deeper Lagoon (`#076b67`) for hover and pressed states and a series of translucent Lagoon tints for soft fills, hairlines and focus rings. Four further hues are kept for atmosphere only and never carry meaning: Aqua (`#14b7ad`), Mint (`#96e0de`), Ice (`#dbf0fa`) and Sky (`#0f84b4`). They show up in background washes, the hero sky and the call-to-action panel. The grounds are cool and bright: Canvas (`#f4f9fa`) behind each page, White for cards, Haze (`#edf5f7`) and Haze 2 (`#e2eef1`) for nested blocks and tinted areas, and Line (`#dbe8eb`) for borders and rules. Text runs from Navy (`#0b2530`) for headings and key copy, through Tide (`#46626b`) for body text and Fog (`#87a0a8`) for secondary labels, to Pale (`#b7cacf`) for the faintest marks. Three colours exist purely for meaning: Amber (`#e3922f`) for warnings and margin calls, Leaf (`#2e9d5f`) for healthy figures, and Rose (`#dc4f5a`) for liquidations and losses.

**Shape and depth.** Buttons are pills in sentence case, and the primary one carries a soft Lagoon glow. Cards are white with 24 px corners, a hairline border in Line and one long, very soft shadow beneath; nested blocks inside a card use Haze at 16 px, and inputs are 12 px. Headline figures sit in KPI cards and are set in the display face. Badges are small pills in five tones, `lagoon | amber | leaf | rose | muted`, each a light tint of its colour with matching text.

**Navigation.** The platform header is a pane of translucent glass that blurs whatever scrolls beneath it. Across its centre runs a pill-shaped nav on a Haze track, and the page you are on shows as a white pill raised above the track. On a phone the header nav gives way to a blurred bar fixed at the bottom of the screen with the four pages people use every day: Overview, Borrow, Lend and Positions. On small screens, Explorer and Settings are reached through icon buttons in the header.

**Motion.** Movement is slow and quiet. Washes of Aqua and Ice drift behind sections, cards lift a few pixels on hover, and content fades up into place as a page loads. Nothing bounces or flashes, and every animation is switched off for visitors who ask for reduced motion.

The shared building blocks (`.card`, `.surface`, `.btn` and `.input`) are defined in `src/app/globals.css` and used by both applications. For the broader identity, see [Brand and channels](../resources/brand.md).

## Settings and environment
Setup instructions are in `supabase/README.md` and in the `.env.example` file at the root of the repository. The variables fall into three groups:

* Privy: `PRIVY_APP_SECRET` and `NEXT_PUBLIC_PRIVY_APP_ID`.
* Supabase: `NEXT_PUBLIC_SUPABASE_URL`, the service key `SUPABASE_SERVICE_ROLE_KEY`, and the browser key `NEXT_PUBLIC_SUPABASE_ANON_KEY`.
* Chain: `NEXT_PUBLIC_CHAIN_ID=4663`, together with a name, an RPC URL and an explorer.

When a public variable is absent, the platform shows a setup notice naming the missing variable rather than breaking silently.
