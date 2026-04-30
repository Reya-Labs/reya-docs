# Architecture (at a glance)

<figure><img src="../.gitbook/assets/image (50).png" alt=""><figcaption></figcaption></figure>

A based-rollup works through a model comparable to proposer-builder-separation (PBS). Simply, Ethereum L1 validators delegate execution and sequencing rights to a third party. In PBS they delegate to builders. In a based-rollup they delegate to gateways, which we refer to as “Execution Nodes”.

This delegation means Execution Nodes know they will be able to include their transactions in the L1 block. Since this is known, they can they give lightening fast ‘pre-confirmations’ back to traders who are executing trades through the exchange operated within an Execution Node. As a result, Reya has 2 confirmation levels:

1. Pre-confirmed: sequenced by the Execution Node and an execution receipt is available to the user
2. Finalised: the transaction has been batched and included in an L1 block

A based-rollup must post transaction data through to the Ethereum L1. However, posting order data (from the orderbook) would be uneconomical, since order data is orders of magnitude greater than executions data. To solve this problem Reya has a hybrid DA structure:

* Trade Executions data goes to Ethereum L1
* Order data goes to EigenDA

Both are verifiable via ZK-proofs.

The outcome for users is:

* A lightening fast execution environment, achieving sub-millisecond trade speed
* A highly secure structure
  * Ethereum L1 validators are the ultimate owners of block building
  * ZK-proofs allow for full verifiability of the entire trading system, from orders to executions
  * Execution nodes can be slashed if they misbehave, creating economic security
* Synchronous composability with Ethereum, the biggest DeFi ecosystem in the world.

For full details please read the whitepaper here: [https://x.com/0xSimonJones/status/1944737748381839671](https://x.com/0xSimonJones/status/1944737748381839671)
