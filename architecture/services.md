---
description: The off-chain pieces around Spatial's contracts (the relayer, the indexer, keeper bots, the KYC provider and the platform), any of which an outside operator can run and none of which has authority over a loan.
---

# Off-chain services

You could turn off every service described here and no loan would move. That property is intentional. Funds and the authoritative record sit in the contracts, while these services exist so people can read that record quickly and respond to it quickly. Each one is open source, or can be operated by anybody, or holds no power, and usually it is all three at once. They are gauges for the crew to watch, never the controls that steer.

## The web app
* Built with Next.js and viem, and hosted at onspatial.org under the `/platform` path.
* Privy signs users in and gives every one of them an embedded Robinhood Chain wallet. That wallet also produces the EIP-712 signatures on offers and requests, which costs nothing in gas.
* Supabase stores the loan registry together with the shared book of orders, with row-level security keyed to the Privy user.
* The six screens are overview, borrow, lend and positions, followed by explorer and settings. On the lend screen, a user sees the order book next to the offers they have made.

The platform keeps no keys and cannot approve anything. Any number on screen can be cross-checked with the indexer or read straight from the chain. The [platform page](platform.md) explains how it is put together.

## The relayer
The relayer publishes the open book, meaning every signed lend offer and every borrow request that has not yet turned into a loan.

| | |
| --- | --- |
| Design | No state of its own; exposes REST and WebSocket APIs |
| Stores | Borrow requests and signed lend offers |
| Validates | Signature, expiry, nonce status, the maker's balance and allowance (or reserve balance held in the vault), whether each party is eligible, and whether the term and tokens are compatible |
| Serves | Views filtered on token, tier, term, APR and LTV, plus a suggested syndicate for each request |
| Authority | None at all. When a loan originates, the hub checks each offer again on-chain. |
| Operators | Spatial Labs runs one instance and anybody else can run their own. Since the book consists only of signed messages, it can always be reconstructed from nothing. |

Routes over REST and WebSocket:

```
POST   /v1/offers                  post a signed lend offer
DELETE /v1/offers/:hash            soft delete; only the on-chain nonce cancel is binding
GET    /v1/offers?token=&tier=&term=&maxApr=&minLtv=
POST   /v1/requests                post a borrow request
GET    /v1/requests/:id/matches    proposed syndicate for a given request
GET    /v1/book/export             full dump of all live offers and requests
WS     /v1/stream                  live feed of changes to offers and requests
```

The export route is a permanent openness pledge. Anybody can download the entire book whenever they like and run a copy of it.

## The indexer
The indexer derives state from emitted events. It runs either on Ponder (TypeScript, self-hosted) or on Envio HyperIndex against chain 4663, and it has four responsibilities:

* powering the Explorer, Telemetry and the borrower and lender dashboards;
* acting as the source of truth that the keeper bots rely on;
* keeping a copy of the relayer's book, so the order book survives if any one relayer goes offline;
* offering a GraphQL endpoint open to arbitrary queries.

## Keeper bots
Keepers are open-source bots that anybody is free to operate. Where the protocol pays liquidation fees, the operator collects them.

| Bot | Trigger | Action |
| --- | --- | --- |
| Margin alert | Health factor on a loan falls below 1.10 | Messages the borrower through whichever channels they set up |
| Auction opener | Health factor below 1.00, or a loan in Defaulted, while no sequencer grace applies | Calls `startAuction` and receives the keeper share |
| Rollover acceptor | A rollover auction's rate climbing to a lender's limit | Sends in the acceptance that lender signed in advance |
| Reserve rebalancer | Vault liquidity shifting, or a lender updating their preference | Moves funds into or out of the vault through the adapter |
| Oracle watch | A move cap being tripped, a pause flag going up, or a stale price | Passes the pause on to the platform and the alert channels |

## Identity checks
An outside firm of the Sumsub or Persona type performs screening, and a zero-knowledge route is also available. Its steps are:

1. verify identity, sanctions status and residency;
2. write an EAS attestation to the user's wallet, limited to the role they qualify for;
3. refresh or withdraw that attestation each time the user is rescreened.

Only three things go into the attestation: the role, the class of jurisdiction, and an expiry date. Personal data stays off-chain. For how the contracts consume it, see the [Eligibility page](eligibility.md).

## Alerting
Borrowers sign up for alerts with the keeper service through email, a webhook, wallet push, or any mix of these. Notifications cover health factor warnings, upcoming maturity, activity in a rollover auction, any pause of the market, and each parameter change that affects one of their loans.
