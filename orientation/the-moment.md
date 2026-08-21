---
description: Why 2026 is the year a syndicated credit market for tokenized real-world assets became possible, told through the three conditions it depended on.
---

# Why now is the right time

A market like Spatial needed three conditions together. Tokenized stocks had to become a normal on-chain asset carrying a price contracts could rely on. There had to be genuine borrowing demand against RWAs, at meaningful size. And existing lending designs had to leave room that a different design could occupy. In 2024 not one of these held. Today every one of them does.

## What pooled lending cannot do

Almost all on-chain lending against RWAs runs through pools, such as Aave Horizon, Morpho vaults and Kamino. Pools carry three structural limits that parameter tuning will never fix:

* Every borrower is priced off one utilisation curve, regardless of the collateral posted.
* A bad loan is paid for by all the pool's suppliers together.
* Assets lacking deep liquidity have to be tightly capped or excluded.

None of that bothers treasuries or index ETFs. It all bothers a holder of one mid-cap stock token who wants credit for 30 days at a rate a particular lender would accept. What that holder needs is pricing per loan, a term that ends, and risk that remains with the party who chose it. Spatial is built to be exactly that, as the page on [the eight design rules](eight-rules.md) explains one rule at a time.

## July 2026: the obstacle removed

Robinhood Chain launched its mainnet on 1 July 2026. It is the only place where a regulated broker issues tokenized stocks as ordinary ERC-20s on an L2, backed one to one, with Chainlink feeds pricing them 24/5, plus account abstraction under ERC-4337 and contract deployment open to anyone. Robinhood describes these tokens as suitable for "lending pools and trading collateral". The longer argument lives in [why Robinhood Chain](../chain/the-case-for-robinhood-chain.md).

## How big tokenized stocks have become

Excluding stablecoins, transferable RWAs on public chains stood near $7.9B at the end of 2024, near $21B at the start of 2026, and near $38.7B now. Tokenized stocks are the fastest growing category in that figure, going from around $424M in mid-2025 to around $2.59B in August 2026. Sources vary by roughly 20 percent, largely because they disagree on counting assets that cannot be transferred.

| Category | Transferable value on-chain | Behind the figure |
| --- | --- | --- |
| **Tokenized stocks** | **~$2.6B** | Ondo Stocks over $1B TVL; xStocks over $25B of cumulative volume; issued by Robinhood, Coinbase and Binance |
| Tokenized US Treasuries | ~$16.0B | USDY $2.2B, BUIDL $2.8B, BENJI ~$2.4B, USYC $2.9B |
| Tokenized credit | ~$7.5B distributed | Roughly $35B once "represented" assets that cannot be transferred, Figure HELOCs for instance, are counted |
| Commodities | ~$3.1B | XAUT, PAXG |
| Real estate | ~$175M | |
| VC and private equity | ~$1.6B | |
| Stablecoins | ~$303B | The settlement layer underneath every other row |

Source: rwa.xyz, data as of 28 August 2026.

## Demand to borrow against RWAs

Across DeFi, total deposits fell roughly 15 percent over the year. RWA deposits into DeFi lending went the other way and tripled, reaching $7.4B (CoinShares, Q2 2026).

* **Aave Horizon**: by mid-2026, around $540M supplied and $163M borrowed against treasury and CLO tokens, with access permissioned.
* **Morpho**: RWA deposits rose from almost nothing to around $400M during 2025, reached roughly $875M on Avalanche by March 2026, and $80M to $90M is already deployed on Robinhood Chain.
* **Solana stock-token collateral** (Kamino, Jupiter Lend): around $53M by late July 2026. The figure is low because of structure, not appetite. Pooled markets are forced to keep single-stock exposure tightly capped.

For scale, US margin debt on its own is above $900B, and securities-backed lending worldwide runs into the trillions, nearly all of it supplied to wealthy clients by private banks and prime brokers.

## Lessons from earlier RWA credit

The last cycle showed plainly what fails: unsecured loans to borrowers no one could examine, covenants a chain has no means to enforce, concentration in a handful of obligors, and wrapper prices that could be manipulated. Spatial only writes overcollateralised loans, secured by liquid on-chain collateral with an oracle price. Recovery is a contract-run liquidation rather than a court claim. Each episode is examined in [the precedents the design responds to](../risk/precedents.md).
