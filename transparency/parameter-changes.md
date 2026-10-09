---
description: How a Spatial parameter change moves through the RiskConfig timelock, which settings governance controls, and which things lie permanently outside its reach.
---

# Changing a parameter

None of Spatial's core contracts can be upgraded. The only thing governance can adjust is the set of values stored in `RiskConfig`, and each adjustment follows the same path: announced, held for a delay, executed, recorded. The guardian's pause is the single exception.

## What governance controls

| Within reach | Permanently out of reach |
| --- | --- |
| Haircuts, exposure caps and tier LTVs | Swapping out or upgrading core contract code |
| Staleness bounds, move caps and oracle adapters | Escrowed collateral and lender principal |
| Auction curves and rollover settings | Pausing repayment, or the release of collateral once repaid |
| Fee rates | Altering the terms of a loan that is already live |
| The lists of attestation issuers and whitelisted vaults | Minting $SPATIAL, which has a fixed supply |
| An emergency halt on new originations and liquidations | Acting before the timelock delay has elapsed |

## The process

1. **Draft the proposal.** It sets out the change and the reasons for it, supported by backtesting figures or by data from Telemetry, the public risk page.
2. **Announce and schedule.** A call to `schedule(calls, salt, rationale)` from the owner multisig queues it, with `rationale` set to the hash of the published explanation. `OperationScheduled` captures the batch, the earliest time it may execute and the hash. The log displays the current value, the proposed value and the reasoning.
3. **Let the delay pass.** After bootstrap the delay is at least one hour, and it runs in full so that anyone affected has time to read the change and respond. A queued batch can be withdrawn with `cancel`.
4. **Execute.** After the delay ends, the multisig has a 14 day grace period in which to call `execute(calls, salt)`. Each setter then emits the event `ParamChanged(key, subject, value)`, identifying which setting changed, what it targets (a tier, a token or an address), and the new value in ABI encoding.
5. **Leave the record.** On the Explorer, the governance log keeps the proposal, the reasoning behind it and the transaction that executed it, permanently.

Any batch not executed within its grace period expires and has to be scheduled again.

## Effect on existing loans

* Origination and rollover fees use the rate current when they become payable, so a fee that has already been paid is never affected by a later change.
* The interest share is looked up in `RiskConfig` when a payout happens, so a change governs payouts made after execution.
* When a tier is demoted, existing loans only move to the new liquidation LTV after a grace period, announced alongside the demotion, has ended. That gives borrowers room to respond.
* Changes to oracle adapters apply immediately, since their purpose is to correct.

## Emergency pause

The guardian holds exactly one power, `setPaused`. It stops liquidations and new originations, emits `EmergencyPause`, and has no effect on repayments or top-ups. Only the owner can lift it. The pause is meant for oracle incidents, newly found vulnerabilities and events at the chain level, and after each use a rationale goes into the log and an incident note is written.

## Bootstrap and ownership

`RiskConfig` begins in a bootstrap mode in which the owner sets values directly while a deployment is being wired together. Calling `finishBootstrap()` closes that mode permanently.

Ownership sits with a timelocked multisig controlled by the foundation, whose signers come partly from the team and partly from independent parties. Handover takes two steps: `transferOwnership`, then `acceptOwnership`. The roadmap moves parameter control from the multisig to a governance module, which keeps the timelock and the public rationale, still has no path to funds, and still cannot pause repayment. More in [the foundation and the operating company](../compliance/entities.md).
