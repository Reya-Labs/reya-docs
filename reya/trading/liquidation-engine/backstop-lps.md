# Backstop LPs

The backstop LP pool would be a pool where people deposit funds which can be used to force positions onto them. It is important to not that the liquidation process never pushes losses onto backstop LPs, as positions, as this mechanism is only activated while possible losses can be covered by the insurance fund. The true drawback for backstop LPs, then, is not having to absorb liquidation losses, but rather the need to actively manage the positions that were pushed onto them so as to avoid subsequent losses while holding the portfolio.

In return for providing liquidity to backstop the liquidation process, backstop LPs get (at least) the following benefits:

1. a portion of collected liquidation fees is distributed to backstop LPs.
2. Exemption from liquidation fees when a position is pushed onto them, regardless of whether it is at profit.
3. the system will actually try to push the positions at profit: when we reach say 25% of LMR, the positions would be pushed onto backstop LPs at bankruptcy price, that is, the entirety of the margin would be transferred (as opposed to the earlier stage, where the Protocol is trying to minimize a discount). This will generally be below market price.
4. In any case, the worst that can happen is that a position gets pushed at market (TWAP) price if the account’s losses have exhausted the margin balance, and then only below the ADL threshold.
