# Cross-collateralization

REYA creates a new type of margin account that allows both for:

* posting of collateral in any token, so that users may leverage their preferred tokens while holding on to them. As example use cases, users can hold onto their yield bearing tokens (e.g., stETH) while backing a portfolio in their base token (e.g. ETH); or holding ETH to back a USDC position, perhaps ensuring a constant collateralization ratio by posting ETH to back a short USDC-denominated ETH perp.
* Aggregation of portfolios in different settlement tokens, unprecedently. This is a unique feature in Reya, which allows users to share PnL across portfolios in different denominations, analogously to the sharing of PnL with a settlement token. By doing so, users can automatically virtually borrow from one portfolio onto another with actual conversion happening only as needed, ensuring maximal synergy.

Reya’s cross-collateralization module uses a system of bubbles and haircuts based in USDC to manage collateral mismatch. This means that:

* when converting amounts from one token into another, a haircut is applied to conservatively account for possible future movements of the exchange rate;
* margin requirements are first netted out, as much as possible, within each bubble, to avoid distorting effects of the haircuts;
* closely associated tokens (e.g. yield bearing tokens) are included at the bubble level to minimize unnecessary haircuts;
* all surplus or deficit margin in the bubbles is then rebased and netted off.

The following diagram illustrates the system, where we use USD as the root of the diagram as an example.

<figure><img src="../../.gitbook/assets/image (32).png" alt=""><figcaption></figcaption></figure>
