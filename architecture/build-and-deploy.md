---
description: How to compile and test Spatial's contracts, the sequence the deploy script follows, the environment variables it consumes, Robinhood Chain specifics worth knowing, and the file that records the live addresses.
---

# Building and deploying

The contracts live in the repository's `contracts/` folder as a Foundry project. `foundry.toml` pins the compiler to Solidity 0.8.26, targets the Cancun EVM and turns on via-IR. Dependencies are pulled with soldeer rather than as git submodules: forge-std 1.9.7 and OpenZeppelin 5.2.0.

```bash
cd contracts
forge soldeer install
forge build
forge test -vv
```

## Which addresses to trust

`contracts/deployments/4663.json` is the single source of truth for what is deployed on chain 4663, Robinhood Chain. Those same addresses appear in the platform's `deployment` parameter row, and the hub's address is shown in the Explorer's parameters panel. Treat any address missing from that file as untrusted. Source code for each deployment is verified on Blockscout. For more, see [Network parameters](../chain/parameters.md) and [Reading the Explorer](../guides/explorer.md).

## Tests

Running `forge test` executes 139 tests in eight files, organised as nine suites. They share `Base.t.sol`, a fixture that stands up the whole stack against mocks and produces signed offers and requests.

| File | What it exercises |
| --- | --- |
| `Origination.t.sol` | Three lenders funding one loan; the LTV limit per tier; borrowers and lenders who lack eligibility; an APR over the borrower's ceiling; offers that were cancelled; a single standing offer that funds two loans; relayed requests; exposure caps; a guardian pause; drawing on reserve capital; replayed requests; the cap on slices; gated slice transfers; sequencer grace; overflow protection |
| `Repayment.t.sol` | Interest accruing slice by slice; a third party repaying; the minimum interest period; repaying in part and in full; collateral top-ups; repaying in the grace window, while new loans are paused, and once a rollover has failed |
| `LiquidationAuction.t.sol` | How price drives the health factor; a complete Dutch auction through to the proceeds waterfall; buying in pieces; who may liquidate; default after grace runs out; sequencer grace; a paused oracle; stale or quiet feeds depending on session; the closed-market haircut, its floor and lender opt-outs; checkpoints taken for the move cap; payouts in kind; requiring a stream report; pausing and restarting an auction that has stalled |
| `RolloverMarket.t.sol` | Who can open a rollover and when; the rate rising linearly; clearing with a new syndicate; failing into default; offers priced above the current rate; cancellation by the borrower; capacity handed back after a failure or a liquidation; the cap on acceptances |
| `RiskConfig.t.sol` | Bootstrap and the timelock; batches that go stale; the limits of the guardian; validating parameters; attestations expiring and being revoked; removing an issuer; gated slice transfers; withdrawals from the fee vault; wiring that can happen only once |
| `OracleGuard.t.sol` | Setting up feeds and the cases it refuses; the quote token; rounds from the sequencer feed; stream reports whether fresh, stale, missing a timestamp, dated in the future or divergent; prices that round to zero after rescaling |
| `Reserve.t.sol` | Depositing to the reserve and withdrawing with yield; guarding against overdraws; access restricted to the hub; delisting a vault without trapping anyone's funds |
| `Invariants.t.sol` | An invariant over the loan ledger driven by a handler, along with fuzzing of partial repayments, splits between syndicate members, and liquidations |

The wider testing picture, covering fuzzing, mutation testing, fork tests, formal verification and audits, is on the [Assurance](assurance.md) page.

## Shipping to a network
Start by copying `.env.example` to `.env`. The minimum is `PRIVATE_KEY` and `ROBINHOOD_RPC` (docs.robinhood.com/chain lists the official RPC URLs). On a live network you also need `USDG`, `STOCK_TOKEN`, `STOCK_FEED` and `VAULT`. Then run:

```bash
source .env
forge script script/Deploy.s.sol --rpc-url robinhood --broadcast --verify
```

That single invocation handles the whole deployment in the following sequence:

1. Creates `RiskConfig`, `PermissionRegistry`, `CredentialRegistry`, `OracleGuard`, `SliceNote`, `FeeVault`, `CreditHub`, `LiquidationAuction` and `RolloverMarket`.
2. While the risk config is still in bootstrap mode, writes the opening settings. These are: `CredentialRegistry` acting as eligibility adapter; who may issue attestations; three tiers numbered 1 to 3 (shown to users as A to C); an initial collateral token with its exposure cap; USDG enabled for lending; allowed terms of 7, 14, 30 and 90 days; parameters for loans, auctions, rollovers and fees; and the whitelisted vault.
3. Registers the collateral feed with the guard, and adds Chainlink's sequencer uptime feed if one is supplied.
4. Creates `ReserveAdapter`, then calls `SliceNote.setDesk`, `ReserveAdapter.setDesk` and finally `CreditHub.wire(liquidationAuction_, rolloverMarket_, reserve_)`.
5. Calls `finishBootstrap()` when requested, and begins the two-step ownership transfer to the governance multisig, which finishes it by calling `acceptOwnership()`.

The resulting addresses are saved to `contracts/deployments/<chainId>.json` with these keys: `chainId`, `RiskConfig`, `PermissionRegistry`, `CredentialRegistry`, `OracleGuard`, `SliceNote`, `FeeVault`, `CreditHub`, `LiquidationAuction`, `RolloverMarket`, `ReserveAdapter`, `StockToken`, `StockFeed`, `USDG` and `Vault`. Every seed script reads its addresses from this file, and so does the platform.

### Environment variables

| Variable | Used for |
| --- | --- |
| `PRIVATE_KEY` | The deploying wallet, which owns `RiskConfig` until `GOV` accepts it |
| `ROBINHOOD_RPC`, `ROBINHOOD_TESTNET_RPC` | RPC URLs that the `robinhood_testnet` and `robinhood` aliases resolve to |
| `USDG`, `VAULT`, `STOCK_TOKEN`, `STOCK_FEED` | USDG as the loan token, an ERC-4626 vault holding reserve capital, then the tier 1 collateral and the Chainlink feed that prices it |
| `SEQUENCER_FEED` | Address of the Chainlink L2 Sequencer Uptime Feed; left out if empty |
| `MARKET_STATUS_SOURCE` | Contract with a `marketStatus()` function used to classify sessions; if empty, the guard falls back to its built-in weekly timetable |
| `STREAM_ADAPTER` | Points at the adapter that auction pricing uses to reach the Data Streams verifier; if empty, only Data Feeds are used |
| `STALENESS_CLOSED`, `STALENESS_EXTENDED`, `STALENESS_REGULAR` | Maximum price age for each session, in seconds; unless set they are 345600, 7200 and 3600 respectively |
| `MOVE_CAP_WINDOW` | How many seconds the checkpoint window lasts for the move cap of 25 percent; 3600 unless set |
| `GOV`, `GUARDIAN` | The governance multisig and the holder of the pause key; each falls back to the deployer |
| `TIMELOCK_DELAY` | In seconds, 3600 unless set. After bootstrap ends it can never again drop below 3600 |
| `ATTESTATION_ISSUER` | The KYC signer allowed to write attestations; falls back to the deployer |
| `FINISH_BOOTSTRAP` | Set to `true` to end bootstrap as the final step, so every subsequent change must pass through the timelock |
| `DEPLOY_MOCKS` | Set to `true` to let mocks fill in for any dependency left empty; on by default for every chain other than 4663 |
| `BLOCKSCOUT_API_KEY`, `ROBINHOOD_TESTNET_BLOCKSCOUT_API`, `ROBINHOOD_BLOCKSCOUT_API` | The key (its value is `unused`) and the endpoints for source verification |

### Other scripts

`DeployCore.s.sol` is a slimmer variant intended for testnets. It brings up `RiskConfig`, `CreditHub` and `LiquidationAuction` with what they depend on, and leaves out `RolloverMarket`, `ReserveAdapter` and `FeeVault`. The seed scripts give a deployment some activity. `Seed.s.sol` and `SeedDeep.s.sol` work against a full deployment; `SeedDeep.s.sol` cuts the timelock delay to one minute so that `SeedExecute.s.sol`, launched about a minute afterwards, can carry out a queued change and put the one hour delay back. `SeedCore.s.sol`, `SeedMore.s.sol` and `SeedResume.s.sol` play the same part for a core-only deployment.

## Robinhood Chain testnet (46630) notes

* `CreditHub` is roughly 125 bytes below the EIP-170 ceiling of 24,576 bytes, so check `forge build --sizes` whenever the hub changes. Beyond the ceiling, deploying only works with `--code-size-limit 1000000` (or `--disable-code-size-limit` on a local Anvil fork). If either flag becomes necessary, the right fix is to slim the hub down rather than keep the flag around.
* Source verification runs through Blockscout instead of Etherscan. Either add these flags when deploying, or verify afterwards by appending them to a resumed run:

  ```bash
  --verify --verifier blockscout --verifier-url https://explorer.testnet.chain.robinhood.com/api/
  # after the fact, add as well:
  --broadcast --resume --private-key $PRIVATE_KEY
  ```
* Add `--slow` and `--gas-estimate-multiplier 200`. Because L1 calldata is charged as additional gas units, the limits forge works out from simulation fall short on the heavier calls. Doubling the estimate and sending transactions one by one gets them through.
* Set `GOV` so the run finishes by offering `RiskConfig` to the multisig, and set `FINISH_BOOTSTRAP=true` so that every later change waits on the timelock.

## Local development

When targeting Anvil, leave all dependency addresses empty. In that case the script puts mocks in their place: USDG (six decimals), an `NVDAx` Stock Token, an ERC-4626 vault, and a Chainlink aggregator whose answer is 176.40 with eight decimals. If the deployer is also the issuer, it also grants the deployer four roles: `RELAYER`, `LIQUIDATOR`, `BORROWER` and `LENDER_PROFESSIONAL`. From there a loan can be taken through its whole life:

1. Fund a lender with mock USDG and a borrower with mock Stock Tokens, and have each approve `CreditHub`.
2. Produce a `LendOffer` signature over EIP-712 typed data whose domain has the name `Spatial` and version `1`, uses whatever chain id Anvil reports, and names `CreditHub` as verifying contract.
3. As the borrower, call `originate(request, "", [offer], [signature])`.
4. Look at `debtOf`, `healthFactor` and the balance held in `SliceNote`.
5. Then either call `repay`, or push the price down with `MockAggregatorV3.set()` and begin an auction through `LiquidationAuction.startAuction`.

Mocks never form part of a production deployment. Once the contracts are live, [The contract set](contracts.md) explains the job of each one.
