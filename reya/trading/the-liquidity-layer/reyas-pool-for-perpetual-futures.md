# Reya’s pool for perpetual futures

Reya Network’s initial deployment already includes a passive liquidity mechanism for perpetual fututes. It uses a novel constant-product pegging mechanism for perpetual futures. It powers any exchange operating on Reya Network, and allows for unprecedented capital efficiency, both for traders as well as LPs. For the first time ever on-chain:

1. Traders can leverage positions with full portfolio cross-margining
2. LPs can offer capital at collateralization ratio of less than 100% of nominal in addition to cross-margining

Reya Network’s passive pool mechanism accomplishes this by introducing a series of new safety mechanism designs, such as:

1. a constant product pegging mechanism to the spot market, a novel price impact function based on the constant product curve, which, uniquely, automatically adapts to changes in liquidity while incentivizing rebalancing of the pool’s exposure;
2. a dynamic funding rate that, also based on the constant product pegging, further strongly incentivizing rebalancing of the pool’s net exposure by using the price impact function to determine the funding rate velocity;
3. a margin system of Reya’s Derivatives Clearing mechanism, allowing for wide control of the risk profile while also enabling full portfolio cross-margining;
4. an open trading system, enabling developers to build their exchanges on top of the instrument and clearing systems, further incentivizing liquidity and trading in Reya Network.

At heart, the constant-product-pegged passive liquidity mechanism is a market making model that provides the best features of a traditional constant product AMM while allowing LPs to offer price-passive liquidity in a highly efficient manner. This means that LPs can offer liquidity without continuously pricing crypto assets while fully leveraging their capital.

To enable price-passive LPing, the passive LP mechanism makes full use of the spot market prices, which can be used to index the futures. It also aggregates all liquidity, creating a unique interface between makers and takers.

Critically, the pool only acts as a temporary counterparty to trades, by offsetting trade flow automatically and freeing up capital for new trades. Finally, the price impact function based on the constant curve not only allows the pool to automatically adjust to changes in liquidity conditions, but also strongly disincentivizes liquidity drainage.
