---
description: The services and protocols on Robinhood Chain that Spatial integrates with, arranged by where they meet a loan, and what each one is relied on for.
---

# Integrations on Robinhood Chain

Much of the infrastructure a lending market needs already existed on Robinhood Chain when Spatial launched. The protocol connects to it instead of building it again. Below, each integration is sorted by the point where it meets a loan: along the route from origination to liquidation, at the user's side, and in the background.

## Along the loan's route

### Chainlink for prices and market status

Data Feeds and Data Streams deliver equity prices 24/5 with the multiplier already applied. The Sequencer Uptime Feed triggers the grace period after an outage. Once ACE and CCID are generally available on chain 4663, they could sit behind the permission registry. The [pricing and sessions](../risk/pricing.md) page has more.

### Paxos for the loan currency

USDG is issued natively on the chain and falls under MiCA. All loans are priced, funded and repaid in it, and co-marketing with Paxos is part of the go-to-market plan. See [USDG as the loan currency](../collateral/usdg.md).

### Morpho Blue for idle capital and auction bidders

Morpho runs Robinhood Earn and holds about $80M to $90M in RWA deposits on this chain. Lender USDG that has not been matched yet waits in a whitelisted Morpho USDG vault and is withdrawn as soon as a loan is created. Arbitrageurs and liquidators who already work on Morpho are obvious bidders in Spatial's liquidation auctions.

The two protocols do not compete. Morpho offers variable-rate loans against deep, liquid collateral, while Spatial offers fixed, negotiated rates against long-tail collateral. More in [Idle capital between fills](../mechanics/parked-capital.md).

### Uniswap and 1inch for market depth

The chain hosts Uniswap v2, v3 and v4 along with 1inch. How deep the DEX market is for a Stock Token feeds into its tier and its exposure cap, and arbitrageurs working those pools should be the most active bidders when liquidation auctions run.

## At the user's side

### Smart accounts

Both EIP-7702 and ERC-4337 entry points work on the chain, and Alchemy Gas Manager and ZeroDev supply the infrastructure. Spatial's platform handles signing through Privy, covering sign-in, embedded wallets and gas-free EIP-712 signatures, so lenders can post offers at no gas cost. A single user operation can bundle approve and borrow, and smart accounts produce signatures via EIP-1271. See [Platform architecture](../architecture/platform.md).

### Robinhood Wallet

This is the main wallet for borrowers. The platform deep-links to it, and getting listed in its dApp directory is among the distribution aims.

### TRM Labs

TRM provides sanctions and AML screening in two independent layers. It screens the entire chain at the sequencer, and the KYC provider that backs Spatial's attestations screens users when a credential is issued and again each time it is renewed.

## In the background

### Blockscout

Each entry in a flight log, the event history shown per loan in the Explorer, links through to its transaction on Blockscout. See [Using the Explorer](../guides/explorer.md).

### Indexing for the Explorer and keepers

Loans, slices and events in the Explorer come from the platform's Supabase registry. For dashboards and keeper bots that read chain 4663 directly, two indexers have been evaluated: Ponder (self-hosted, TypeScript) and Envio HyperIndex. See [Off-chain services](../architecture/services.md).

### Arbitrum

Because Robinhood Chain is an Arbitrum Orbit chain, Spatial can apply for Arbitrum Foundation audit subsidies, the Arbitrum Open House 2026 buildathon track and Chainlink BUILD, and it plans to go after all three.
