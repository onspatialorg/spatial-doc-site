---
description: How Spatial decides which wallets can borrow, lend, liquidate or relay, what a credential records, how the permission registry responds, and which calls revert on a refusal.
---

# Who may take part: credentials and roles
Every permission check in the protocol comes down to a single view function:

```solidity
function isEligible(address account, bytes32 role) external view returns (bool);
```

Stock Tokens themselves do not restrict their holders. Robinhood screens people where they enter its app and at the KYC'd venues that mint and redeem the tokens, which leaves the token free to move on chain. Any protocol that lends against it therefore needs its own perimeter, and Spatial builds that perimeter from this one function. If a wallet holds no current credential for the role a call demands, it cannot fund or open a loan, join a rollover, bid in a liquidation auction or receive a slice token, and the call reverts before any tokens change hands. Reads are ungated, so aggregators, explorers and indexers can view the full book without holding any role.

## Roles in use

Each role is a `bytes32` key. `Permissions.BORROWER` equals `keccak256("spatial.role.BORROWER")`, and the remaining four follow the same pattern.

| Role | Granted to | Availability |
| --- | --- | --- |
| `BORROWER` | KYC'd wallets in none of the restricted jurisdictions (the same list that applies to Stock Tokens). Today the holders are verified businesses, wealthy individuals who meet the high-net-worth test, and professional clients. | Live |
| `LENDER_PROFESSIONAL` | Clients treated as professional under whichever regime governs them, for example credit funds, market makers and family offices | Live |
| `LENDER_RETAIL` | Retail lenders, switched on one market at a time where regulation permits | Roadmap |
| `LIQUIDATOR` | Addresses that may purchase auctioned collateral, or receive it when a loan settles in kind | Live |
| `RELAYER` | Services that send a borrower's signed request on their behalf | Live |

## Calls that check a role

| Function | Role required, and of whom |
| --- | --- |
| `CreditHub.originate` | `BORROWER` for the borrower named in the request; `RELAYER` for the sender if it is someone else; either lender role for each offer's maker |
| `RolloverMarket.accept` | Either lender role for whoever made the incoming offer |
| `CreditHub.clearRefinance` | Either lender role for every member of the new syndicate, checked once more as their slices are minted |
| `LiquidationAuction.buy` and `buyWithLimit` | `LIQUIDATOR` for the buyer |
| A `SliceNote` transfer from one holder to another | Either lender role for the recipient. The hub's own mints and burns are not checked |

Two flows deliberately skip the check. `repay` returns escrow to the borrower recorded on the loan, and nothing is allowed to block that transfer. A lender paid in collateral was already checked when their slice was minted, and again on any transfer of it.

## What the permission registry returns

`PermissionRegistry` contains no rules of its own. Each query is handed to the adapter named by `RiskConfig.eligibilityAdapter()`, which is looked up fresh on every call rather than cached, so a governance change takes effect on the next query. If no adapter has been set, the call reverts with `AdapterNotSet`. The registry also offers `isLender` and `isBorrower`, the reverting checks `requireRole` (which fails with `NotEligible`) and `requireLender` (which fails with `NotLender`), and `attestationOf`, which returns a record's uid and expiry.

## Contents of a credential

A wallet earns a role through a credential that the KYC provider issues once identity, sanctions and residency checks have all passed. Its format follows the schema of the Ethereum Attestation Service and is compatible with ONCHAINID. It records:

* the wallet address;
* the role granted;
* a jurisdiction class (the actual country appears only if the holder chooses to disclose it);
* an investor class, where one applies;
* the date it expires.

That is the full list. There is no name, no document and nothing else that could identify the person. Credentials lapse on their expiry date and are refreshed through periodic re-screening, while a revocation applies from the block in which it is recorded.

A wallet can also use a zkPass or Privado ID proof instead. Such a proof states that the wallet qualifies under jurisdiction class X and investor class Y, and discloses neither the provider that ran the check nor any of the underlying attributes. The registry treats that proof and a credential identically.

## The bundled CredentialRegistry

The deploy script installs `CredentialRegistry` as the adapter. It is an EAS-shaped store maintained by Spatial for chains that lack EAS. Writes are limited to issuers approved in `RiskConfig`, and that approval is checked against the risk config on every call, so removing an issuer applies immediately and invalidates all of its records.

Records are written with this call, which emits `Attested`:

```solidity
function attest(address account, bytes32 role, bytes2 jurisdictionClass, uint8 investorClass, uint64 expiry);
```

The expiry has to lie in the future (otherwise the call fails with `BadExpiry`), and a second attestation for an identical wallet and role pair overwrites the earlier record entirely. `revoke(account, role)` flags the record as revoked and emits `Revoked`. Since any approved issuer can revoke any record, a sound issuer can tidy up after one whose keys were compromised. `isEligible` returns true only when the record exists, has not been revoked, has not expired and comes from an issuer that is still approved. The class codes belong to the issuer: they are kept for indexers and the protocol never reads meaning into them.

## Changing the backend

Since the registry does nothing but delegate, its source of truth can be replaced without changing any contract that relies on it. Candidates include:

* an **EAS adapter**, finding attestations by schema and issuer;
* an **ONCHAINID adapter**, for identities that follow ERC-3643;
* a **Chainlink ACE / CCID adapter**, available to governance as an additional policy engine.

`RiskConfig` holds both levers: the issuer list, changed with `setAttestationIssuer`, and the active adapter, changed with `setEligibilityAdapter`. Trimming the issuer list or switching provider is therefore a timelocked, logged parameter change and never needs a redeployment; [How parameters change](../transparency/parameter-changes.md) explains the process. The platform mirrors these records into an `attestations` table so that a wallet can see its own roles on the settings page, as covered in [the platform overview](platform.md).

## Tokens that carry their own transfer rules

Should Stock Tokens or bridged RWAs gain ERC-3643 or ERC-7943 (uRWA) hooks, the escrow will call `canTransfer` and `canReceive` before any movement and will itself be added to the issuer's allowlist. Both sets of rules then apply together, and neither has to trust the other. More detail is in [Collateral still to come](../collateral/next-markets.md).

## Four independent barriers for restricted jurisdictions

A wallet from a restricted jurisdiction has to get past four separate layers, none of which depends on the others:

1. Credentials are never issued to wallets from restricted jurisdictions.
2. At onboarding, every wallet declares where it is based.
3. The front end geo-fences by IP address and blocks restricted jurisdictions.
4. The sequencer runs its own sanctions screening.

Full details are on [Jurisdictions](../compliance/jurisdictions.md).
