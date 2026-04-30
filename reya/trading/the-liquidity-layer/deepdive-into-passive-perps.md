# Deepdive into passive perps

The perpetual futures give traders to gain leveraged exposure to a reference market by paying a funding rate. Each market is determined as an instance of the passive perp instrument determined by an oracle. Like any ‘cash-settled’ derivative, it is full determined by its cashflows:

$$
\mathrm{PnL}=\mathrm{priceVariation} + \mathrm{fundingFee}
$$

## Price variation

For Reya’s passive perps, the intermediate price variation (before position closing) are computed using the _spot_ price given by the oracle, consistent with the pegging. This means that at any time, for each unit of exposure the account’s unrealized PnL is

$$
\mathrm{spotPrice}-\mathrm{entryPrice}
$$

where $$\mathrm{entryPrice}$$ might be an average of actual entry prices, or also of prices at the last PnL realization.

In Reya perps, this mark to market due to price variation is credited initially to the account’s unrealized PnL. This is so that the system keeps profits on escrow for a while, in case an extreme event makes an auto-deleveraging (ADL) necessary: ADL computes bankruptcy prices on the basis of unrealized PnL (see the documentation for the Derivatives Clearing Protocol).

## Dynamic funding rate

Rather than a traditional funding rate mechanism, Reya Network uses a **dynamic** funding rate, introduced by Synthetix, adapted to the constant-product pegging. In a dynamic funding rate, the price deviation determines not the funding rate itself, but its **velocity**.

Details of how the funding rate velocity is set can be found below. But if $$\dot{r}$$ denotes the current funding rate velocity, and the original funding rate is $$r_0$$ at time $$t_0$$, the funding rate at subsequent time $$T$$ is

$$
r_T=r_0+\dot{r}\Delta^d(t,T)
$$

Here, $$\Delta^d(t,T)$$ is the time interval between  and $$T$$ measured in _days._ The convention for traditional perps is to express the time interval in hours, in which case a factor of $$1/24$$ must be included.

In the absence of ADL events, the funding fee accrued by a single exposure token while the velocity remains constant is

$$
\int_{0}^{\Delta^d(T_1,T_2)}p_t\left(r_{T_1}+\dot{r}_{T_1}\Delta^h(T_1,t)\right)d\Delta^h
$$

where $$p_t$$ is the spot price. This could be approximated by querying the spot price at regular interval, but this is not practical given the smart contract limitations. Instead, the accrued funding is approximated as

$$
p_{T_2}\left(r_{T_1}+\dot{r}_{T_1}\Delta^h(T_1,t)\right)\Delta^h(T_1,T_2)
$$

This accrued amount is added every time the funding rate velocity changes, and so the accuracy of this approximation depends on the frequency of transactions and so should it is expected to be high. On the other hand, the use of $$p_{T_2}$$ means that the actual amount is not known until the end of the interval, and so intermediate queries are estimations only — but the frequency of transactions should make this sufficiently accurate.

## The constant-product pegging mechanism

### The constant-product pegging formula

Reya’s constant-product pegging mechanism takes into consideration:

1. the spot price
2. the available liquidity in the pool;
3. the pool’s net exposure.

The pegging is ‘constant-product’ because it uses the slippage implied by the constant product curve known, for example, from Uniswap and other AMMs. It immediately adjusts trade prices in response to any of these three variable. Concretely,

$$
\mathrm{tradePrice}=\mathrm{spotPrice}\times \left( 1-\frac{\mathrm{netExposure}}{\mathrm{maxExposure}+\mathrm{netExposure}}\right)
$$

Here, $$\mathrm{netExposure}=\mathrm{longOI} - \mathrm{shortOI}$$ is the pool’s net exposure _**after**_ the trade.

<figure><img src="../../.gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>

Notice a couple of important features of the pegging formula:

1. if there is no net exposure (i.e., the long and short sides are perfectly balanced), the quoted price is actually the spot price;
2. however, whenever previous trades cause an imbalance, the price gets adjusted in a way that disincentivizes further unbalancing, and does so increasingly stronger.

This price deviation is _approximately linear_ when the trade volume is much smaller that the maximum exposure, but, as it approaches the limit, the constant product effect kicks in and it quickly shoots up. The following table illustrates this effect for a maximal exposure of 100,000.

| Units:          | 0 (spot) | 60      | 600     | 6,000   | 60,000 |
| --------------- | -------- | ------- | ------- | ------- | ------ |
| Price:          | $2000    | $2001.2 | $2012.1 | $2127.7 | $5,000 |
| Deviation (bp): | 0        | 6       | 60      | 638     | 15000  |

Uniquely, in Reya Network’s constant-product-pegged passive liquidity mechanism the price impact function automatically adjusts _both_ trade prices and the funding rate in response to liquidity changes. This optimizes trading conditions to the risk profile of the pool at any moment: increasing the price impact and the funding rate if liquidity were drained; but also improving trading conditions in response to increases in liquidity both from new deposits and accumulated PnL.

### The maximal exposure

Because the pool is not a spot exchange, but rather trading in ‘cash-settled’ derivatives, the maximal exposure is a measure of capital consumption by margin requirements. The capital locked for the existing exposure is expressed as a Protocol-set multiple $$\lambda$$ of the initial margin requirement $$\mathrm{IMR}$$. The pool’s capital would be exhausted if

$$
\mathrm{totalCapital}=\lambda\mathrm{IMR}
$$

since it could take on more positions. Maximal exposure is so defined that this is never verified as trading the pool to this level would set an infinite trade price.

The maximal exposure for a given market is the maximum exposure that would result in the pool’s exhaustion, _holding all other exposures constant_. Holding the other exposures constant might both have a positive or negative effect in the maximal exposure, because cross-margining might actually offset the net exposure in a market with those of others.

### Funding rate velocity

The net exposure of the pool determines the funding rate velocity through:

$$
\dot{r}^{hourly}=\alpha \times \frac{1}{24}\frac{\mathrm{tradePrice}-\mathrm{spotPrice}}{\mathrm{spotPrice}}=-\alpha \times \frac{1}{24}\frac{\mathrm{netExposure}}{\mathrm{depth}\times\mathrm{maxExposure}+\mathrm{netExposure}}
$$

where $$\alpha$$ is a scaling parameter. If the original funding rate is $$r_0$$ at time $$t_0$$, the funding rate at subsequent time $$T$$ is

$$
\begin{align*} r_T&=r_0+\dot{r}\Delta^h(t,T)\\ &=r_0-\alpha \Delta^d(t,T) \times \frac{\mathrm{netExposure}}{\mathrm{depth}\times\mathrm{maxExposure}+\mathrm{netExposure}} \end{align*}
$$

where the superscript in $$\Delta^h$$ denotes time measured in days, while $$\Delta^d$$ in days.

**Relation to traditional funding rates:** If nothing changed, under a traditional funding rate with a recovery period of 24 hours, we would incentivize convergence between the two prices by charging, every one-hour period, the amount

$$
\frac{1}{24}\frac{\mathrm{tradePrice}-\mathrm{spotPrice}}{\mathrm{spotPrice}}=\alpha \times \dot{r}\times \frac{1}{24}
$$

In other words, one hour after a change in net exposure, if the funding rate was at zero and the scaling parameter is one, the velocity will add precisely the amount that the normal funding rate would charge. However, a dynamic funding rate will keep _increasing_ until the pool is rebalanced (or its net exposure changes sign).

