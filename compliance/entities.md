---
description: Spatial Foundation owns governance and the contracts, while Spatial Labs builds and operates the platform. The reasons for separating them, and how control of parameters moves from the multisig to on-chain governance.
---

# The two Spatial entities

Two separate entities stand behind Spatial, each with a distinct role, and the protocol is built so that it would continue if either one went away.

## Reasons for the separation

* The contracts cannot be changed and take custody of nothing. Placing them with a foundation keeps governance independent of the developer's commercial interests.
* The front end deals directly with users and, in certain places, must be licensed. The operating company takes that on, and keeping it apart narrows the regulatory exposure that attaches to the protocol.
* All the code is open source. Should the operating company shut down, anybody could run a relayer, an indexer and a front end of their own, and the contracts would keep working exactly as before.

## Who does what

| | Spatial Labs (operating company) | Spatial Foundation |
| --- | --- | --- |
| Owns | The platform plus the reference relayer, the indexer and keeper software | Protocol governance and the immutable contracts |
| In charge of | Running everything off chain from day to day | The timelocked parameter multisig, until a governance module replaces it |
| Staff and licences | Employs the team, and holds a licence wherever one is needed | No staff and no licences |

## Governance now and later

**Now.** The foundation's multisig, sitting behind a timelock, makes all parameter changes. Every proposal is published along with the reasons for it.

**Later.** A governance module takes over parameter control. The handover keeps four guarantees intact: the timelock stays, rationales are still published, there is still no route to user funds, and nothing gains the power to pause repayment.

## Where $SPATIAL sits
The website lists the address of $SPATIAL, which is deployed on Robinhood Chain. The supply is fixed, and governance cannot mint any more. The complete change procedure is set out in [How parameters change](../transparency/parameter-changes.md).
