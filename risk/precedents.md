---
description: What went wrong at Maple, Goldfinch, Centrifuge and Edel Finance, the numbers involved in each case, and the Spatial rule built in response.
---

# Lessons from four failures

Moving a loan on chain alters the way it settles, but it does nothing for the borrower's ability to repay. From 2022 through 2026 several protocols lost sight of that distinction, and the cost is well documented. Spatial reads that history as a set of requirements, and each of the four cases here corresponds to a specific rule.

## Four losses, three rules

Every one of these incidents comes down to one of three decisions: making loans without security, relying on enforcement that takes place off chain, or putting a value on something other than the asset really held. Spatial's design excludes all three from the start.

1. **Every loan is overcollateralised.** Collateral must be liquid, held on chain and priced by an oracle.
2. **Recovery happens in smart contracts.** The auction proceeds with no need for anybody to agree or cooperate.
3. **Price only what sits in escrow.** No wrapper is ever accepted, and the protocol runs its own checks on top of Chainlink feeds.

With those three in place, the risks left over are the issuer, gaps while markets are closed, the sequencer, and the code. Each is covered, with its control, elsewhere in this section.

## The incidents

### Maple Finance, December 2022: the Orthogonal Trading default

| | |
| --- | --- |
| Size of the loss | Around $36M, close to 30% of Maple's total lending |
| Sequence of events | Orthogonal Trading borrowed from Maple's pools with no collateral, misrepresented how much it had exposed to FTX, and failed to repay after FTX went under. |
| What it teaches | Never make unsecured loans to a counterparty whose books you cannot inspect. Maple today operates at roughly 165% collateralisation, with collateral kept in custody. |
| How Spatial responds | A loan exists only once escrowed collateral exceeds the debt. Borrowers are never asked for disclosures, since no part of the system relies on them. |

### Goldfinch, 2023 to 2026: borrower defaults

| | |
| --- | --- |
| Size of the loss | Roughly $18M across three borrowers (Tugende, Stratos and Lend East). GFI fell 99.8%, and the platform went into maintenance mode in June 2026. |
| Sequence of events | The emerging-market borrowers were held only by covenants enforced off chain. When they defaulted, recoveries came slowly and fell short, with Lend East repaying around 42%. |
| What it teaches | No blockchain can enforce a covenant. Getting money back off chain means legal proceedings lasting years. |
| How Spatial responds | Recovery is a Dutch auction lasting roughly 45 minutes, which any address may start. There is nothing to enforce, since the contract is already holding the collateral. |

### Centrifuge, 2023: ConsolFreight and Harbor Trade

| | |
| --- | --- |
| Size of the loss | Around $4M, counting MakerDAO's share |
| Sequence of events | The trade-finance pools depended on a handful of obligors. Once they failed, redemptions were frozen for months. |
| What it teaches | Concentrated exposure is deadly, and a frozen pool traps every participant together. |
| How Spatial responds | Caps on each token keep concentration low. Slices are independent, so a failing position locks up nothing but itself. With no pool and no queue for redemptions, there is nothing shared to freeze. |

### Edel Finance, July 2026: the wGOOGLx exploit

| | |
| --- | --- |
| Size of the loss | Around $403K of bad debt |
| Sequence of events | The Chainlink price of tokenised Google shares stayed accurate the whole time. What went wrong was a wrapper token whose exchange rate was pumped 78 times and then treated as collateral value. |
| What it teaches | Value exactly the token you are holding. Any derived rate opens an attack surface. |
| How Spatial responds | `OracleGuard` reads the feed for the token in escrow and nothing else. Wrappers, vault shares and derived rates are never accepted as collateral. The [pricing page](pricing.md) explains how. |
