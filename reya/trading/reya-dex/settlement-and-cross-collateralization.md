# Settlement and cross-collateralization

### The perps on Reya DEX settle in rUSD. What does that mean?

To settle in rUSD means that all prices and PnL payments are denominated in rUSD. So, for example, if you follow the price of ETH on the Reya dApp and see it price go up from 3500rUSD to 3550rUSD, that means shorts pay longs 50rUSD. Funding payments and fees are also paid entirely in rUSD.

_A direct consequence of this is that your margin requirements are also denominated in rUSD._ Margin requirements, after all, are funds locked by the Network to ensure you have enough funds to pay for possible losses.

### Does that mean I need to hold rUSD in my account to cover my margin requirements?

Only partially so. **One of the killer features of Reya DEX is that enables&#x20;**_**cross-collateralization**_**:** you can hold some tokens other than rUSD in your margin account, and have them count towards your margin requirements. For example, you can cover them instead with a yield bearing collateral: (s)USDe or deUSD. This way, your collateral can earn yield while you trade.

**However, you still have to pay your losses in rUSD.** The perps, after all, still settle in rUSD! In other words, you should still keep a minimum amount of rUSD in your account, otherwise you may be subject to asset liquidation procedures\[link].

### So, my account can hold a variety of tokens, but my margin requirement is still rUSD. How does that work out?

That’s where your _**margin balance**_ comes in: it’s the rUSD-denominated amount used to assess your margin coverage. It converts the value of any token into rUSD using oracle prices (using the same ultra fast Stork data feeds that the perps use), then compares the total amount to your margin requirement. The margin balance is the balance figure that you can find in the trading screen.

### Why does my margin balance show up lower than the value of the assets in my account?

Although you can cover your margin requirement with a variety of tokens other than rUSD, their value is taken into account with a _haircut_, i.e., their value is reduced by a percentage that depends on the particular token.

Haircuts cover for the risk that your tokens reduce in rUSD value by the time they are needed to cover losses. This would leave your account insolvent. You can think of the haircut as a sort of margin requirement in reverse: it’s the amount of rUSD Reya can safely credit your margin balance after taking price fluctuations and volatility into account.

If you want to know the actual value of all your collateral without haircuts, the number you are looking for is the a**ccount value** or **equity**, which you can find in your portfolio dashboard.

### If not all the collateral value is taken into account _and_ it runs the risk of liquidation, why would I use cross-collateralization?

Cross-collateralization is useful for many reasons, but there are two very big use cases:

1. **accrue yield while trading:** if you deposit a yield bearing token into your margin account, that token will still be accruing yield even while you are using it to trade!
2. **boost capital efficiency and reduce liquidation risk:** for example, the PnL of an asset is (approximately) matched by the price variations of that asset. Using that asset as collateral will then minimize value fluctuations in your account, it being _market neutral_ while accruing the funding rate.

_Remember that perps still settle in rUSD!_ This means that the cross-collateralization can optimize your capital deployment, but you still need to carefully manage your rUSD balance to avoid collateral liquidations.
