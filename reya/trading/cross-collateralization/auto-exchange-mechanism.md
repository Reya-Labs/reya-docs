# Auto-exchange mechanism

When one of the auto-excange conditions is breached, the system will forcefully convert other tokens into T by relying in external ‘auto-exchangers’ who receive a discount over market price as incentive. The amounts available will take into account both the amounts in the T-bubble, as well as those that can be moved from other bubbles.

The amount of collateral $$m$$ of token $$S$$ the auto-exchanger receives in exchange is computed as

$$
(1-\mathrm{discount}_{S/T})X_{S/T}\times m=n
$$

where both $$\mathrm{discount}_{S/T}$$ and $$X_{S/T}$$ will generally be a compound term, computed as products/quotients of the official pool discounts and exchange rates (one needs to trace the S→T path on the directed graph).

The following diagram illustrates the general flow.

<figure><img src="../../.gitbook/assets/image (33).png" alt=""><figcaption></figcaption></figure>
