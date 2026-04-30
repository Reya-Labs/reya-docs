# Liquidation Engine

Liquidations are the procedures through which Reya cuts positions off to stop accounts from accumulating losses larger than their margin balance. They are triggered whenever the liquidation margin requirement (LMR) is breached for a bubble. In the presence of cross-margining, for a given settlement token $$T$$ we need to verify the LMR both at the ‘local’

$$
\mathrm{NDelta}_T=\mathrm{BBal}_T-\mathrm{LMR}_T\leq 0
$$

as well as the ‘global’ level:

$$
\mathrm{NDelta}{USD}=\sum{S_{T'}>0}(1-\mathrm{haircut}T)X{T'/USD}S_{T'}\leq 0
$$

where . Essentially, even if a bubble cannot satisfy the LMR autonomously, we check if there are funds in the other bubbles that can be auto-exchanged.

## The liquidation waterfall

_Reya Network introduces a series of liquidation features that guarantee the health of its collateral pool for the first time on-chain._ These are all gathered in an innovative waterfall structure, each lower layer being deployed as the account’s health worsens:

1. **Open bidding**, where in anticipation of LMR breach, liquidators submit liquidations proposals that get filtered, ranked and executed once LMR actually is breached, where the ranking takes into account both the requested discount and the liquidity profile of the requested assets, and is designed to disincentivize over-liquidations;
2. **Dutch auction**, where liquidation bids are immediately accepted at a given margin reward (risk-weighted discount), and which is activated if the open bidding was insufficient to bring an account back to health, or if the LMR was breached directly.
3. Pushing positions onto backstop LPs, who provide a liquidity pool against which to push the portfolio when the margin balance falls below a threshold level;
4. Auto-deleveraging, where losses are socialized: the system tries to act early enough so that only other users’ potential PnL will be curtailed (by pushing positions at market prices with insolvencies covered by the insurance funds), but will also socialize losses in last resort (by pushing positions at the bankruptcy price.

In other words, the waterfall serves as a series of backstops to ensure that the account can be fully liquidated while preserving the pool’s health.

The important design choices are:

* By introducing the concept of maintenance margin as distinct from liquidation margin, Reya Network allows for price optimization while maintaining speedy liquidations once the accounts threaten insolvency.
* Liquidations within Reya always work through position transfer to particular parties, in practice outsourcing the optimization of unwinding cross-margined exposures to liquidators: the Protocol is able to organically grow in scope as portfolios and liquidators become more sophisticated.
* Positions are always transferred by referencing the price index: as explained above, this ensures the coherence between margin requirements, marking to market, and liquidation losses.
* Untrusted liquidators are compensated with a margin balance (’cash’) transfer proportion to the reduction in LMR. In this way, no untrusted liquidator can cause the account to lose more than the LMR, which should prevent different types of attacks. Also, the liquidator’s reward being in ‘cash’ (rather than a discount on price index) should incentivize them, since it means their liquidation profits are always immediately realized.

<figure><img src="../../.gitbook/assets/image (35).png" alt=""><figcaption></figcaption></figure>
