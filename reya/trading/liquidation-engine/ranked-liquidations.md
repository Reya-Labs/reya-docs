# Ranked liquidations

The first level is the open bidding process, during which bidder will submit a series of volumes $$\{Q_i\}$$ that they are willing to take on from the liquidated account, as well as a percentage of the resulting change in LMR that they request as reward. The open competitive process should ensure that the reward parameter is optimized.

<figure><img src="../../.gitbook/assets/image (34).png" alt="" width="563"><figcaption></figcaption></figure>

The other cornerstone for this procedure is a scoring function used for liquidation bids, which summarizes the reward parameter and the market impact. For a bid requesting volumes $$\{Q_i\}$$ of $$n$$ assets for a reward parameter $$d$$, the score is

$$
\mathrm{score}(\{Q_i\},d) = \frac{1}{n}\left(w(1-d) + (1-w)\frac{\sum_i Q_i \times\mathrm{pSlippage}i\left((Q_i(Q{i,max}-Q_i))^{1/2}\right)}{\sum_i Q_i}\right)
$$

where $$\mathrm{pSlippage}_i(Q)=\phi Q^\beta/\mathrm{price}_i$$ is the estimated percentual price slippage (using governance set parameters).

An important feature is of this scoring function is that it disincentives over-liquidations by multiplying by $$1/n$$ (which in practice stratifies bids according to the number of requested assets), as well as maxing out the slippage benefit at half the available volume.
