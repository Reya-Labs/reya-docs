# Perpetual Futures

Reya Exchange is a perpetual futures exchange built on top of Reya Network. It will display all of the features unlocked by the financial logic of Reya Network. In particular, it provides both traders and LPs with unprecedented capital efficiency by:

1. **offsetting losses from one position with profits another**, and enabling leveraging of all portfolio profits immediately;
2. **offsetting and reducing margin requirements,** by recognizing the P\&L offset that can happen between positions.

All of this is accomplished with all the security of on-chain settlement and clearing. All your capital is guarded safe by a smart contract, and there is no “FTX-risk” that someone will just run away with it.

### Perpetual Futures <a href="#perpetual-futures" id="perpetual-futures"></a>

Perpetual futures are derivatives contracts where traders can have a PnL matching that of a given token without actually buying that token. For example, a trader that holding a long position for a unit of ETH will have the same PnL that they would have if they actually held one unit of ETH: when the ETH price goes up from 3,000rUSD to 3,100rUSD they will earn 100rUSD; on the other hand, if the price goes down to 2,900rUSD, they will have to pay 100rUSD. These amounts are interchanges with accounts holding short positions, in other words, traders that earn when the price goes down and vice versa.

Additionally to the PnL from price variations, longs and shorts will also interchange a funding rate. This funding rate is used to incentivize balancing of long and short open interest, and to keep the perpetual futures in line with the spot market.

When a trader enters a perpetual futures contract, they need to deposit collateral to ensure that they will be able to cover losses that they suffer in their positions. Managing these collateral requirements, settling PnLs, as well as resolving accounts when a trader has an insufficient balance is the main work of Reya’s Derivatives Clearing Module.
