# Context

## Building Ethereum’s execution layer&#x20;

Settlement and execution are very distinct components of financial trading, each with its own specific challenges as well as security requirements. Because of this, monolithic L1s that try to satisfy both functions force false trade-offs that ultimately limit the growth and development of a truly decentralized financial system. These false trade-offs fall into the general ‘performance vs decentralization’ theme.



At Reya, we strongly believe that the future of decentralized finance passes through Ethereum. Ethereum is unparalleled in terms of the security and decentralization assurances it offers as a settlement layer. Furthermore, it has benefited from the wide participation of contributors in various areas of expertise, resulting in a flexible and cutting-edge roadmap that incentivizes innovation across all areas of DeFi and crypto in general.



Based rollups have been an important recent development. Rollups are key components in scaling Ethereum’s. They precisely offer the possibility of a divide-and-conquer approach to application/execution and settlement. However, L2s in their current form risk being centralized systems that aren’t credibly neutral and reliable long-term. This is the problem that based rollups tackle head-on: they allow for true expansion of Ethereum’s capabilities while inheriting its security and decentralization guarantees.



[Based rollups—superpowers from L1 sequencing - Layer 2 - Ethereum Research](https://ethresear.ch/t/based-rollups-superpowers-from-l1-sequencing/15016)



By being a based-rollup, Reya will specifically become an execution layer tailored to the microstructural properties of financial trading, bringing execution to unparalleled performance levels while still providing traders with full Ethereum assurances on their capital.

<figure><img src="../.gitbook/assets/Screenshot 2025-11-19 at 11.39.19.png" alt=""><figcaption></figcaption></figure>

In a based-rollup, Ethereum L1 validators ultimately control transaction ordering. By anchoring sequencing to validators who restake ETH, Reya strengthens liveness and censorship resistance versus single‑sequencer designs while keeping the system tightly aligned with Ethereum.



Ethereum validators can operate or delegate to high-performance nodes under a lookahead schedule that maps L1 slots to execution node identities, providing predictable and sticky leadership windows. Transactions follow two key confirmation levels: pre‑confirmed (execution receipt issued) and finalized (batched to L1).



The rotating execution nodes process trades and issue millisecond pre-confirmations. They run Reya’s rust-based trading engine, which is cross-compiled into zero-knowledge circuits enabling asynchronous proof generation and independent stateless validation of state transitions.



Orders and executions have different bandwidth needs, motivating Reya to adopt a hybrid data availability (DA) setup where account state updates are published to Ethereum using EIP‑4844 blobs, while high‑throughput order data leverages a specialized external DA provider.



Execution misbehavior is deterred via slashing and provable discrepancies between signed pre‑confirmations and on‑chain data. Notably, Ethereum validators will also face penalties if the execution node they delegated to is slashed.
