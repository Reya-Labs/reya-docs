---
hidden: true
---

# Automatic leveraging of PnL across instruments

Even in the basic token-matched case above, $REYA unlocks capital efficiency by automatically leveraging all PnL into a user’s balance. In practice, this means that all PnL is automatically part of a user’s balance. Explanation of all terms follows below:

$$Bal_T = \mathrm{netDeposits}_T + \mathrm{realizedPnL}_T + \mathrm{unrealizedPnL}_T$$

This feature is sometime known as cross-margining, although the term is more appropriately used for offsetting of margin requirements, also offered in Reya as explained below.

One consequence is that gains from any position can offset losses in other positions, an in particular hedged positions have their PnL reflect their hedging. Furthermore, any PnL that accrues beyond the initial margin requirement can be used and truly leverage into new positions. All of this is essential to incentivize liquidity provision, since LPs accumulate large hedged portfolios and the PnL leveraging reflects that. It is also a big incentive for traders, of course.

The following diagram illustrates the automatic leveraging of PnL when instruments are separately margined with a single margin account.



<figure><img src="../.gitbook/assets/image (31).png" alt=""><figcaption></figcaption></figure>
