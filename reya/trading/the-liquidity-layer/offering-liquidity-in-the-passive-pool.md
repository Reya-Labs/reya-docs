# Offering liquidity in the passive pool

LPs can provide funds to the pool via a segregated margin account dedicated to providing liquidity to the passive mechanism, distinct from a participants trading margin account. Once users make a deposit, they are entitled to the proportional returns of the pool.

Direct returns of the pool can come from four different components:

1. Trade spreads: as the pool quotes prices based on the net exposure after the trade, long-short trades that immediately follow each other generate a spread;
2. Funding fees: as the funding rate velocity depends on the pool’s net exposure, the funding fees _eventually_ are in the pool’s favour, and so the pool accrues funding most of the time. The exception is when net exposure changes sign, and the change in the velocity takes sometime to take effect and to change the funding rate sign as well.
3. Protocol fees: the fees generated from the trades are shared with LPs.
4.  Backstop PnL: if the pool is designated as a backstop LP for the passive perps, it derives two other sources of PnL:

    1. a portion of the insurance fees derived from liquidations;
    2. profits from taking on the liquidated positions.

    Note that backstop LPs always take on positions at a favorable price, and never at a loss; the insurance fund or ADL are triggered if needed.

As with any investment, providing liquidity to the passive pool includes risks:

1. Although the mechanism design works to make aggregate trading profitable, not all trades are closed at a profit;
2. Although the dynamic funding rate responds to correct persistent imbalances, the pool could in an extreme case be locked into positions, rather than follow the typical pattern of opening and closing exposure;
3. Extreme insolvencies in the protocol could trigger an auto-deleveraging event, which could hit the pool;
4. Even if the pool is profitable, the capital in it might be locked to collateralize exposure and therefore LP withdrawals might be restricted to keep the pool well-collateralized. Again, this is an extreme circumstance that the mechanism design works to both disincentivize and correct if it ever happens. As soon as the pool rebalances, LP funds are again automatically unlocked.
