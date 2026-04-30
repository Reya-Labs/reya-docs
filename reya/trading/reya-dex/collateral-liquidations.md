# Collateral liquidations

### I less have cross-collateral than I deposited. What happened?

If your balance of collateral tokens other than rUSD was reduced, you were subject to collateral liquidation. This can happen for two reasons:

1. your rUSD balance dropped to zero, and the collateral was needed to cover further losses;
2. your account approached LMR (Liquidation Margin Requirement): your collateral is progressively converted to rUSD to prepare for liquidation procedures (not least, to fix and maximize the amount of rUSD available in your account).

### What exactly is collateral liquidation?

Collateral liquidation, also known as auto-exchange, is a forceful conversion of a token into rUSD. This conversion uses the same oracle prices as the margin balance, but:

* a **percentual penalty** is included in the price, both to disincentivize collateral mis-management, as well as reward the liquidator who ultimately takes on the price risk of the liquidated collateral;
* **liquidation fees** are also charged, as a percentage of the liquidated amount (in rUSD), towards the insurance fund.

This is similar to what happens with position liquidations. The biggest difference for collateral, everything is assessed in terms of the total amount of liquidated collateral, whereas for positions, which imply leverage, reference LMR instead.
