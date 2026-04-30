---
description: What is the passive liquidity mechanism?
hidden: true
---

# The Liquidity Layer

The passive mechanism is an on-chain trading protocol which preserves the best features of the original AMMs:

1. **unique meeting point:** it creates a unique meeting point between makers and takers, aggregating all of the liquidity and generating a single price for every trade;
2. **no liquidity drainage:** it permanently offers liquidity and quotes a price, which allows traders to always trade if they found the price acceptable;
3. **passive LPing:** it allows LPs to make their capital available without having to actively manage their position and keep updating their quote price.

Funds deposited by passive LPs are to bridge between long and short open interests as the pool acts as a temporary counterparty making up the market’s _net imbalance._

<figure><img src="../../.gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>

Passive liquidity pools permanently quote a price for each requested volume by pegging themselves to a reference market. As the Network grows and becomes more robust, the endgame will be to have the passive pools in the Liquidity Layer reference internal markets only, through the use of price indices.

<figure><img src="../../.gitbook/assets/image (27).png" alt=""><figcaption></figcaption></figure>
