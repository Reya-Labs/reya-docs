---
description: Documenting the key external functions of the Reya smart contracts.
hidden: true
---

# Contract Functions

Reya Protocol is separated into 3 main components:

* Core - the margin engine contract which imposes margin requirements and coordinates liquidations, manages margin accounts and their collateral holdings
* Passive Perp Instrument - manages perpetual future markets, pricing of market orders executed against the passive pool, matching as well as keeping track of account exposures and PnLs in available markets
* Passive Pool - liquidity pool which allows anyone to stake rUSD and get pool shares in return (currently pool shares are not available as tokens, but are rather just trackers in the contract). The share price of the pool is dependent on its PnL and fees from trades executed against passive perp traders.
