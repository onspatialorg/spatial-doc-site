---
description: Which parties may lend and which may borrow in the EU and EEA, in the United States and in the other countries where Stock Tokens are offered, with the reasoning behind each stance.
---

# Where Spatial is available

{% hint style="info" %}
This page is not legal advice. Everything described is enforced by the contracts, as explained in [Eligibility and attestations](../architecture/eligibility.md).
{% endhint %}

## Summary by region

| Region | Who can lend | Who can borrow | Rationale |
| --- | --- | --- | --- |
| EU and EEA | Professional clients | Verified companies, and individuals who are professional or high net worth | Spatial's main market |
| United States | Professional clients | Verified companies, and individuals who are professional or high net worth | Access depends on passing verification |
| Remaining Stock Token countries | Professional clients | As in the EU, subject to local rules | Excluded wherever loan intermediation is licensed |

## EU and EEA

Spatial's main market. EU law treats a Stock Token as a financial instrument under MiFID II, and not as a MiCA asset. A peer-to-peer loan with a Stock Token as security is a credit agreement between two parties, and what the platform provides is a service that intermediates loans.

That position rests on three things:

* **Lending is for professional clients.** Because only professional clients can lend, what the lender holds stays outside the rules for offers to retail investors. Retail lenders are admitted country by country, under the local crowdfunding or intermediation framework.
* **Borrowing is for businesses, professionals and high-net-worth individuals.** Consumers as borrowers would trigger national consumer-credit rules, so retail borrowing opens only in places where those requirements are satisfied.
* **USDG is a stablecoin regulated under MiCA**, and that supports its use for settlement across the whole EEA. More in [USDG, the loan currency](../collateral/usdg.md).

## United States

Spatial is open to US persons, who go through exactly the verification everyone else does: the attestation issuer checks identity, sanctions and suitability. Lending is open to professional clients. Borrowing is open to verified businesses and to individuals who are professional or high net worth. At no stage of a loan does the protocol use managerial discretion. Origination, repayment, rollover and liquidation all follow rules, and any eligible party is able to trigger them.

## Other Stock Token countries

Spatial starts from the list of more than 120 countries where Stock Tokens are available, then removes any jurisdiction where loan intermediation needs a licence and doing it unlicensed is forbidden. The attestation issuer keeps the final list up to date, and it is published on the eligibility page.

## VASP, CASP and money transmission

Money stays in escrow at all times and fiat is never exchanged. Despite that, the operating company has a VASP or CASP assessment carried out in every jurisdiction where the platform is made available, and the front end geo-fences any restricted jurisdiction.

## Anti-money-laundering controls

* Each record is screened against sanctions lists when it is created and again when it is renewed.
* Screening is repeated after issuance too, and revoking a record has immediate effect.
* The sequencer runs its own separate screening through TRM Labs, giving an independent second check.
* The indexer and the relayer monitor for patterns in transactions that warrant a closer look.
