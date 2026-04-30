# srUSD

## What is rUSD staking?

**The power of diversification**

**On-chain liquidity provision mechanisms are generally siloed and application specific. Reya’s unified liquidity model makes it possible for capital to be allocated in a diversified manner, improving its risk profile orders of magnitude over what you can find elsewhere in DeFi.**

#### **Shared liquidity**

Reya introduced a model of unified liquidity which allows LP capital to be dynamically allocated to the best opportunity. The AMM model, in particular, relies on a single pool of capital to provide liquidity across all markets, while safely weighing them according to their risk profile. This allows it to provide superior depth, with use cases as a DeFi internalized ‘hedging venue’ with CEX-like execution.

#### **Bridging DeFi and CeFi**

Reya introduced a Liquidity Manager framework, through which Network Liquidity is allocated for different uses through smart contracts or partners. Institutional partners Amber Group and Selini Capital joined efforts with Reya to showcase how this can unlock superior returns for LPs, while also being ‘weaponized’ for Network growth.&#x20;

### Current Managers

Liquidity Managers can be both on-chain protocols or active institutional managers. Their allocation of assets is subject to a clear mandate, although it includes some discretionarity.

### **Allocations**

### Reya’s risk-adjusted returns outperform market benchmarks

Liquidity Managers broadly provide two types of strategy: market neutral ones through cash & carry trades and/or cross-exchange arbitrages; and liquidity provisioning ones (currently, Reya’s AMM supports perp trading on Reya DEX).

#### **Consistently High Returns with minimal drawdowns**

The POC of the unified liquidity framework has proven itself not only resilient, but thriving amid the extreme volatility since its roll out in early December 2024. **Averaging a 10.45% APY, it has had only 6 negative days, with APY over negative days averaging only -0.99% (or a -0.27bp drawdown).**

### Risks associated with rUSD staking

All financial products carry a certain degree of risk, and even low risk investment strategies contain an element of uncertainty. Different instruments involve different levels of exposure to risk and you should be aware of the risks associated with each of these instruments. The information contained in this brochure is a general description of the risks associated with the specific products or services which we may provide to you. You should not rely on the highlighted risks as being the only risks in relation to the product or service. You should always satisfy yourself that a product or service is suitable for you in light of your financial circumstances and that you fully understand the nature and risk associated with that product or service. Any risks highlighted are not to be relied upon as investment advice or a personal recommendation.

#### **Market risk**

Staked rUSD is exposed to market risk, due to the movement of token prices.

* Funding rates are not necessarily positive. Prolonged periods of negative rates would pose a risk to the staked capital. Among other criteria, Reya selects the tokens for this strategy so as to minimize the risk of such episodes.
* While the AMM nets out exposure as the trading flow oscillates between the different sides, it still holds some exposure and market movements may cause losses to the capital. The trading fees compensate for this possibility, as well as the mechanism design which shifts incentives as needed to rebalance the pool.

#### **Smart contract risk**

Staked rUSD is works through smart contracts on Reya Network. Reya smart contracts undergo a rigorous development process with both internal and external audits, however any smart contract has specific risks associated with possible loopholes or bugs in their logic.

A full list of audits can be found at: [https://docs.reya.xyz/technical-docs/audits](https://docs.reya.xyz/technical-docs/audits)

#### **Counterparty risk**

Staked rUSD is subject to counterparty risk in two main respects:

* Funds are apportioned among the Liquidity Managers (Selini and Amber), and therefore their management and solvency might affect the staked rUSD. The delegation of funds to Liquidity Managers is done through legal agreements ensure the segregation of staked rUSD funds from their general capital, and, where possible, manage the allocation of capital through on-chain custodial solutions (in particular, CEFU and Fordefi) to ensure the safety of the funds.
* The strategies employed in staked rUSD imply trades on centralized exchanges, and therefore are subject to counterparty risk of those exchanges. Extreme insolvency of the exchanges could impact the funds locked as margin.

#### **Exchange risk**

All settlement amounts on Reya Network are denominated in rUSD, which is a wrapped version of USDC. This means that capital is at risk in the event of a USDC de-peg v USD.

## Using srUSD to trade

Whenever you stake rUSD, your deposit is _tokenized_. In other words, you receive a new token — srUSD — which represents your deposit. The conversion between these two tokens depends on the price of srUSD, which changes over time according to the returns of Network Liquidity. So if Network Liquidity has a 1% return, then srUSD becomes 1% more expensive; conversely, if Network Liquidity suffers a 1% loss, then srUSD will be 1% cheaper.

<figure><img src="../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

You cannot trade your srUSD, because it is for now non-transferrable. But you _can_ use your srUSD as collateral for trading by moving it to your margin accounts. The margin engine will recognize these tokens and count them towards your margin balance (denominated in rUSD) using the srUSD price _with a haircut_. What this means is that not all the value of your srUSD counts towards the margin balance because of the price risk of srUSD; instead, a small percentage, e.g. 10%, is ignored, and only the rest is counted towards the margin balance. So, in the following example, while your account has a total asset value of 202 rUSD, you can only lock up to 190.90 rUSd as margin for your trading.

<figure><img src="../.gitbook/assets/image (49).png" alt=""><figcaption></figcaption></figure>

Whenever you use any non-rUSD collateral, you need to be mindful of the fact that PnL still settles in rUSD. What this means is that all your profits will be delivered in rUSD, _and so will your losses be paid out._ So, if you suffer losses but you do not have any actual rUSD to cover them, the liquidation engine will forcefully sell a portion of your collateral at a discount.

Given how closely integrated srUSD is with the Network, for srUSD this discount is set to zero. Using a flash swap functionality (which allows for swaps of assets with deferred IM checks), the cross-collateralization system goes and redeems srUSD for the underlying rUSD whenever needed.

You can learn more about cross-collateralization and collateral liquidations [`here`](https://docs.reya.xyz/reya-products/reyadex/trading-on-reya-dex/auto-exchange-and-liquidations).
