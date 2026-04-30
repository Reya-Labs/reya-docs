# How Reya Is Different

Based-rollups are novel mechanisms through which developers can create isolated and performant application specific environments that still leverage the security and consensus of Ethereum L1 validators. These types of rollups haven’t been possible until now; for example some of the key technological unlocks, like the introduction of a [Deterministic Proposer Lookahead](https://eips.ethereum.org/EIPS/eip-7917), have only just got the required R\&D momentum.



Previously, the main successful attempts at creating a performant onchain DEXs have been via

1. An alt-L1
2. A single-Sequencer Zk Rollup

A based zk rollup is superior to these choices in every way:

|                                        | **Single-Sequencer Zk Rollup**                           | **Alt-L1**                                             | **Based Zk Rollup**                                   |
| -------------------------------------- | -------------------------------------------------------- | ------------------------------------------------------ | ----------------------------------------------------- |
| **Speed**                              | Very fast                                                | Fast                                                   | Very Fast                                             |
| **Economic Security**                  | None                                                     | Medium - dependant on L1 assets staked                 | Excellent - Ethereum L1 Proposers and Execution Nodes |
| **Liveness / Reliability**             | Low - Single point of failure                            | Medium - dependant on number of L1 nodes and operators | Excellent - Ethereum L1 Proposers and Execution Nodes |
| **Censorship Resistance / Neutrality** | Low - Single point of failure                            | Medium - dependant on number of L1 nodes and operators | Excellent - Ethereum L1 Proposers and Execution Nodes |
| **Composability**                      | Low - not possible with Ethereum L1 in sychronous manner | Medium - dependant on size of L1 ecosystem             | Excellent - synchronous with Ethereum L1              |

Reya is the first ever trading-specific based zk rollup and a transformational moment for DEX design.
