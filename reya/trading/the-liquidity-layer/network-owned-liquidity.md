# Network Owned Liquidity

Reya Network is a trading-optimised network. This is made possible through a novel network-owned-liquidity design which serves a dual purpose:

1. \[Immediate Term] Providing applications on the network with liquidity
2. \[Medium Term] Acting as economic security of the network

<figure><img src="../../.gitbook/assets/image (42).png" alt=""><figcaption></figcaption></figure>

The primary objective of the liquidity network is to create a mechanism that attracts and retains Liquidity Managers (LMs) who offer the most competitive risk-adjusted returns to Network Stakers (LPs). This acts as a liquidity backbone for ecosystem dApps that in turn bring end-users and order flow into the network discussed in more detail in the following [blog post](https://blog.reya.network/reya-network-phase-2-becoming-defis-liquidity-hub-supercharged-by-connecting-35-trillion-of-offchain-orderflow/).

It is worth noting that a Liquidity Manager can come in various different forms:

1. A passive market-making algorithm
2. An active market-maker allocating capital to enable on-Network or off-Network order flow
3. A passive optimiser across lend/borrow primitives on Reya Network

## Elixir Liquidity Manager

Following [RNIP1](https://reya-network.discourse.group/t/rnip1-the-next-step-for-network-stakers-and-an-early-insight-into-reya-network-v2/10) vote, Elixir is leveraging a share of network liquidity to acquire deUSD and sdeUSD in order to give stakers exposure to yield opportunities from a basis trade strategy. This is enabled via the following adaptations to the pool design.

Reya’s Passive Pool (which acts as a market maker on Reya Dex) is able to support two additional collateral tokens (deUSD & sdeUSD) alongside rUSD. This enables the pool to reallocate some of its rUSD holdings towards deUSD & sdeUSD at the current oracle prices and leverage them as margin in order to market make and boost liquidity available on Reya Dex.

Reya Governance will have the power to decide what percentage of Passive Pool TVL is allocated towards rUSD vs. acquiring deUSD vs. acquiring sdeUSD. Currently pool collateral holding target ratios are set to the following values:

1. rUSD target ratio = 20%.
2. deUSD target ratio = 32%
3. sdeUSD target ratio = 48%

A rebalancer bot(s) is responsible for maintaining the target ratios by exchanging rUSD, deUSD and sdeUSD at the oracle price as long as the makeup of pool holdings deviates from the target ratios beyond a deviation threshold. The deviation threshold is a buffer that is used in order to avoid frequent rebalancing events happening from day to day fluctuations in the pool make up as a result of pool deposits, withdrawals and yield accrual.



It is worth noting that the rUSD denominated value of the passive pool shares is impacted by both the PnL from passive perp market making activity on Reya Dex and changes in the price of deUSD and sdeUSD in rUSD terms. Since sdeUSD is a dollar-yield generating token which is able to capture yield from basis trading activity, reya stakers are able to get exposure to that PnL through an increase in the Passive Pool share price as a result of an increase in the sdeUSD oracle price relative to rUSD.
