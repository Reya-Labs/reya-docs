# Margin

### Margin account <a href="#margin-account" id="margin-account"></a>

A margin account is used to hold and collateralize your derivatives positions. Each margin account can only hold contracts for which sharing of PnL and/or cross-margining is enabled. For example, you will need a specific margin account to hold a position in a contract which is isolated margined.

On the other hand, each of your margin accounts enables:

1. **offsetting losses from one position with profits another**, and enabling leveraging of all portfolio profits immediately;
2. **offsetting and reducing margin requirements,** by recognizing the P\&L offset that can happen between positions.
3. **and, finally, offsetting both P\&L and margin&#x20;**_**across**_**&#x20;exchanges,** by aggregating all positions into a single settlement and clearing layer.

### Margin Balance <a href="#margin-balance" id="margin-balance"></a>

You margin balance are the funds, in rUSD, which you have available to satisfy your margin requirements. Profits will be credited to your margin balance, and losses will be deducted from it. You will be required to have enough rUSD funds to cover the Initial Margin Requirement of your portfolio (in a given account) when opening a new position. You must also keep enough collateral at all times to cover your Liquidation Margin Requirement to avoid being liquidated.

Thanks to Reya’s cross-collateralization features, you can deposit a variety of tokens as collateral. Those tokens will be credited to you margin balance in rUSD terms with a haircut to account for exchange risk: this means that if you deposit 1ETH which is worth 2000rUSD at market prices, you will be credited slightly less than 2000rUSD. **Your ETH will not be automatically converted to rUSD unless you trigger any of the auto-exchange conditions.**
