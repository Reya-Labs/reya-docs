# Auto-exchange conditions

In the first instance, Reya Network’s cross-collateralization features allow users to accumulate a T-specific insolvency for T a settlement token, even while remaining globally solvent. This, however, creates the risk that the pool itself will become T-insolvent if enough of these T-insolvencies accumulate. To avoid this scenario, Reya Network has a set of **auto-exchange** conditions, one set limiting the size of negative T-balances both in absolute and relative terms, and the other set forcing collateral matching (T-solvency) when liquidating.

The auto-exchange conditions can be organized in two groups:

1. **Conditions that limit negative balances:** whereas cross-collateralization allows users to run negative balances, there are three conditions that put limits on this:
   * _Individual token threshold_: for each $$T$$, $$Bal_T \geq -bound_T$$where $$bound_T$$ is a governance set (positive) parameter.
   * _Global threshold_: given all settlement $$T$$ in the margin account, $$\sum_{Bal_T<0} \frac{X_{T/USD}}{1-\mathrm{haircut}_{T/USD}}Bal_T \geq - bound$$, where $$bound$$ is a governance set (positive) parameter.
   * _Negative ratio threshold_: given all settlement $$T$$ in the margin account, $$\sum_{Bal_T<0} X_{T/USD}Bal_T \geq -\eta\cdot \mathrm{AV}$$where $$0<\eta<1$$ is a governance set parameter, and $$\mathrm{AV}$$, the total value of collateral in the account (without haircuts).
2.  **Condition that limits mismatched margin coverage:** whereas the Boss Mode margin coverage condition allows users to cover even the entirety initial or maintenance margin requirements with surplus collateral of other bubbles, once an account breaches the maintenance margin condition, we further require that for every bubble for which there is a deficit maintenance margin the following condition holds:

    $$Bal_T-LMR_T\geq 0$$

    The purpose of this condition is to isolate the liquidation processes of the different bubbles. In particular, the liquidation margin requirement can be separately verified individually for every bubble, and the liquidation process applies always to a single token situation.
