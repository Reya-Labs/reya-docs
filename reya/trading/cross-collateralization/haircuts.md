# Haircuts

A haircut is a percentage decrease in the exchange rate applied for risk reasons: specifically, given two tokens or currencies $$T$$ and $$T'$$, we convert $$T'$$ a collateral not simply by multiplying by the exchange rate $$X_{T'/T}$$, but rather by multiplying by

$$
(1-\textrm{haircut}_{T'})X_{T'/T}
$$

The resulting amount is less than the market value of the  collateral, the difference serving as a cushion for future exchange rate fluctuations that might happen if and until the collateral is converted to cover margin requirements. The haircut, then, can be thought of as margin, but applied statically and in a segregated fashion to the collateral. The methodologies for calculating its values will need risk measures like margin models.

The previous multiplier assumed that the worst scenario is when the exchange rate goes down, which is indeed the case when converting positive quantities. If we convert negative quantities, however, the worst outcome is actually when the exchange rate goes up instead. Therefore, when moving around margin requirements or negative balances, one multiplies instead by

$$
\frac{1}{1-\textrm{haircut}_{T'}}X_{T'/T}
$$

The advantage of this form is that

$$
\frac{1}{(1-\textrm{haircut}{T'})X_{T'/T}}=\frac{1}{1-\textrm{haircut}{T'}}X_{T/T'}
$$

so that the multipliers get interchanges when reversing the sense of the arrows in the graph.
