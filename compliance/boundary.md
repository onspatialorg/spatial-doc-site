---
description: Spatial checks eligibility at the points where value moves into and out of its contracts, using attestations that hold a role and an expiry and no personal information. Here are those entry points, the record each relies on, and three lines the protocol never crosses.
---

# Eligibility at the edges

Spatial's eligibility rules are written into code and applied at the moment a participant interacts with the contracts.

{% hint style="info" %}
This describes how the code behaves. It is not legal advice.
{% endhint %}

## Three lines Spatial never crosses

* User money is held nowhere except the escrow contracts.
* Fiat currency is never exchanged.
* No loan is ever subject to anyone's judgement. Origination, repayment, rollover and liquidation run on rules, and any eligible party may set them off.

These three points support the analysis of money transmission and "managerial discretion" set out on [the jurisdictions page](jurisdictions.md).

## Why the protocol runs its own checks

A Stock Token on chain is an ordinary ERC-20, with nothing restricting transfers. Robinhood checks eligibility inside its app and when tokens are issued or redeemed under KYC, but after a token has left that environment any wallet may hold it. A protocol that accepts it as collateral therefore gets none of Robinhood's checks for free. Spatial holds no custody, so it applies its own screening at every entry point, where `PermissionRegistry` gives the answer. The mechanics are in [Eligibility and attestations](../architecture/eligibility.md).

## Each entry point and its required role

| Who passes | Entry point | Code that checks |
| --- | --- | --- |
| A wallet that has passed KYC and is not in a restricted jurisdiction | Taking a loan | `requireRole(borrower, BORROWER)` when the loan originates |
| A relayer holding an attestation | Submitting a borrower's request on their behalf | `requireRole(msg.sender, RELAYER)` plus the borrower's signature |
| `LENDER_PROFESSIONAL`, or `LENDER_RETAIL` in places that allow it | Supplying funds, both at origination and when a rollover fills | `requireLender(maker)` |
| A liquidator holding an attestation | Purchasing auctioned collateral | `requireRole(msg.sender, LIQUIDATOR)` inside `buy` |
| A lender that has turned on `selfLiquidate` | Taking collateral in kind | The lender role, which the slice has already verified |
| A wallet with a lender attestation | Being sent a slice | `requireLender(to)` within the transfer hook |
| Anyone not located in a restricted jurisdiction | Using the platform at all | Geo-fencing applied by the front end |

## Inside an attestation

**A credential rather than a whitelist row.** Once identity, sanctions and residency have been checked, the KYC provider issues an EAS attestation for that wallet. Within the bundled `CredentialRegistry`, the credential contains a role, a jurisdiction class two bytes long, an investor class, the time of issue and an expiry date. None of these fields can identify an individual.

**Renewal and withdrawal.** Credentials lapse and get renewed by calling `attest` again. A `revoke` applies within the very same block, and dropping an issuer from policy switches off every credential that issuer created.

**Screening twice over.** The KYC provider screens at issuance and again on each renewal. Separately, the sequencer screens on its own account through TRM Labs, which is built into the chain.

**A replaceable rules engine.** The registry itself contains no rules. It forwards every query to the adapter that the risk config points to. The engine can be ONCHAINID, EAS, Chainlink ACE or the bundled registry. Switching it, or changing which issuers are approved, goes through `setEligibilityAdapter` and `setAttestationIssuer`, both behind the timelock, and the core never needs redeploying. That lets stricter regulatory requirements take effect fast.

**Readable by anyone.** All views are public, so explorers, indexers and aggregators have full visibility.

## Personal information

No personal data is written on chain. It is held by the KYC provider. If a wallet wants, it can use a zero-knowledge credential to show it is eligible while keeping hidden both the provider that verified it and every attribute behind the check.

## Tracking the issuer's rules

The set of jurisdictions where borrowers are accepted mirrors the Stock Token eligibility list, so if Robinhood shortens that list, the attestation issuer does the same. If a future version of Stock Tokens adds transfer hooks (ERC-7943 or ERC-3643), the escrow contract will invoke them and will itself be placed on the allowlist. That scenario is followed on [Collateral still to come](../collateral/next-markets.md).
