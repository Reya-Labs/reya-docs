# Auto-exchange and Liquidations

## Position liquidations

### My position was liquidated. Why is that?

Positions are liquidated when they are _undercollateralized_. What this means is that your margin balance went below the minimum level (the _liquidation margin requirement_, LMR) that the margin system considered safe for the exposure you were holding with that position. In other words, too little money to cover for the potential losses you could suffer.

### How can I avoid being liquidated?

For your convenience, the trading app shows the _margin ratio_ of your different positions/accounts. The margin ratio is the ratio of your LMR over your margin balance:

<figure><img src="../../.gitbook/assets/image (43).png" alt=""><figcaption></figcaption></figure>

* If this ratio is below 100%, it means your margin balance is bigger than your LMR, so you are not subject to liquidations.
* However, the margin ratio is constantly changing due market movements, meaning that values under but close to 100% put your account in danger and you should either reduce/close your position voluntarily, or top up your account.
* Once the margin ratio goes over 100%, you account can be liquidated.

### What exactly _is_ a liquidation, anyway?

A liquidation is a forceful closure of positions to reduce or entirely stop the losses your account is accumulating, before your account becomes insolvent. This exposure, however, doesn’t just vanish — there are traders on the opposite side who expect the ‘mirror PnL’ of your position. This means the exposure needs to be transferred to someone else, who receives the position at a discount. This entire operation is handled by a special component called the _Liquidation Module_.

### In addition to the losses from closing my position, I also see money was sent in transfers. What are they for and where do they go?

Whenever an account is liquidated, in addition to having your position closed with a penalty, you will be charged _liquidations fees_. These liquidations fees are proportional to the risk that was removed from your account, and they go to two essential elements of the system: the **insurance fund**, which is a rainy day fund used (and only used) to cover insolvencies; and the **passive pool**, which works as a backstop to all liquidations whenever conditions require it.

### So, basically, liquidations just close my trades at bad prices, and then charge me for it.

Not quite. In fact, in almost all instances, liquidations are only _partial._ The liquidation module will try to close as little exposure as possible, at the least discount possible. Essentially, it tries to bring your LMR down, but just enough to get your margin balance back just below 100%. It accomplishes this by closing only one portion of exposure at a time.

This explains why you might see a series of liquidations in your transaction history: each one closed part of your exposure to bring your account back over water, until markets moved in your favor again, or you are left with no exposure.

### Actually, I can see one big liquidation at a price that is worse than any on the candle graph. And my account has no money in it. What’s up with that?

This means your account was liquidated through the ‘**backstop flow**’. Remember how we said that the passive pool acts as a backstop to liquidations? This happens when the partial liquidations fail to keep your account healthy, or the markets move very quickly against you. Due to the increased urgency in getting your account liquidated, your _entire_ position is pushed directly onto the passive pool, which also takes the _entirety_ of your margin balance as compensation.

The backstop flow is the last resort measure to prevent the **collateral pool** from going insolvent, guaranteeing it can make good on any profits that traders make. The backstop threshold is the margin ratio at which the backstop flow is enabled, and currently set at 125%.

### Does this mean the passive pool is making up my losses?

No. The pool only absorbs positions at a profit, that is, at a price that is favorable. This is why the backstop threshold exists: it’s a level at which the pool can still use the remaining margin balance to create a positions without immediately losing money on it.

So, if you are an LP, rest assured you are not making up any losses from traders. In particular, the passive pool will stop taking on positions as soon as an account’s funds aren’t enough to at least cover the liquidation fees. And remember, the pool gets 50% of fees of _every_ liquidation, not just backstop.

### But then how does the collateral pool stay solvent if my account goes insolvent?

Depending on the circumstances, two things can happen (even both at the same time):

* There is an **insurance fund** which is a rainy day fund that is used (_and can only be used_) to make up any insolvencies resulting after a liquidation. Remember those liquidation fees? That’s what they’re used for.
* In the extreme circumstance that an account were to end up with such a large insolvency that the insurance fund could not cover it, the backstop flow would then activate _auto-deleveraging._\*\* Essentially, as a last resort, the liquidation module will distribute the exposure of the liquidated account proportionally to every trader on the opposing side, partially closing _their_ position.

This last layer of the backstop flow has never been activated, and we hope that is never needed. But it ensures that the collateral pool **always** has enough funds to cover its obligations to traders. And importantly, accomplishes this in the fairest way possible: as unpleasant as it is to have your position closed when you are profiting, _everyone_ gets hit proportionally in the same way. This way, no one can ‘front run’ anyone else on the way out; and no human intervention can tip the scales in anyone’s favor.

Oh! And by the way, _Reya’s the only one DEX that can do this._ Reya is bringing performance _and confidence_ to DeFi.

### What about cross-margined accounts?

Cross-margined accounts are dealt in a holistic way. This means that all positions are liquidated in the same proportion, therefore preserving any hedges that you might have in place. In other words, we take your risk management seriously.

### But wouldn’t it be better to just liquidate the losing positions?

Well… no. If you assume the market is moving directionally (e.g., tanking), then cutting off the losing positions makes sense. But what if the market bounces? Then, the previously profitable positions (which would have been left open) will start losing money when the market reverses, and the liquidated positions won’t be there to profit off the bounce.

The important thing is that we respect any risk mitigations you put in place, including hedges. Ultimately, you should make active risk management an integral part of your trading.

Stay safe out there!
