# Margin System

## Exposures

Instruments in REYA have linear exposures to these risk factors. Explicitly, this means that if risk factors are a multivariate random variable $$R_t$$ of $$n$$ factors, then each instrument determines a vector $$E_t$$such that the price of the instrument is

$$
P_t=E_t'R_t
$$

This means in particular that the entries of $E\_t'$ transform the risk factors into token amounts. As a consequence, note that the exposure vector will often be a function of more than just time: for example, if a given risk factor is the relative (percentual) returns $$r_{t,i}$$ of some asset, then $$e_{t,i}$$ will generally have the price $$p_{t,i}$$ as a factor to transform the percentual return $$r_{t,i}$$ into an absolute return $$p_{t,i}r_{t,i}$$ in token amount. For example, if a user holds exposure of two units of ETH when its price is $1500, then their exposure is $3000.

Finally, note that by linearity, exposures are additive, meaning that a portfolio’s exposure is simply the weighted sum (with portfolio weights) of the exposure vectors of each component.

## The risk matrix

Having established the fundamental framework, pools in REYA take as risk parameters an $$n\times n$$ matrix $$A$$, as well as two multipliers $$\lambda_M$$  and $$\lambda_I$$. Given this matrix, REYA computes the liquidation margin requirement for a portfolio exposure vector $$E$$ as

$$
\mathrm{LMR}=\sqrt{E'AE}
$$

and then $$\mathrm{MMR}=\lambda_M\mathrm{LMR}$$ and $$\mathrm{IMR}=\lambda_I\mathrm{LMR}$$.

The matrix $$A$$of course, must be carefully chosen and updated for the system to appropriately margin portfolios. The cases of multivariate normal and Student's t-distribution are instructive in getting intuition about the risk matrix.

However, REYA will generally rely on a more flexible version, where the components can be independently adjusted according to risk estimates mixing, e.g., Value-at-Risk and Expected Shortfall methods.
