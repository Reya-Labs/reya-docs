# The structure of Reya Network

Right now, Reya Network is composed of two layers.

<figure><img src="../../.gitbook/assets/image (40).png" alt=""><figcaption></figcaption></figure>

## **The chain layer**

This layer optimises the speed and throughput of the network, alongside addressing some important issues on generalisable chains such as front-running and harmful MEV. This layer is powered by Arbitrum Orbit and can achieve 100ms blocktimes and up to 30,000 transactions per second - making it one of the fastest EVM rollups around.

The chain operates with transactions ordered on a FIFO basis - a critical UX improvement for a network optimised for trading. This also removes the ability to front-run and takes away the opportunity for harmful MEV to occur. Overtime, more of the application specific logic associated with trading will be moved into the block-building process itself, driving further improvements to the network.

## **The protocol layer**

The protocol layer directly tackles the vertical integration of DeFi applications by breaking the chain into modular components to support trading, such as PnL settlements, margin requirements, liquidations. It directly provides the components which trading applications can deploy.

This layer is itself composed of multiple parts. Let’s understand each of them in turn:

### **Financial Layer**&#x20;

This is the cornerstone layer that makes holding value, settling payments and trading in a capital efficient manner possible on Reya Network. Importantly, the architecture is open, allowing other exchanges to integrate into the network too.

Reya Network’s architecture has two main modules:

* **Derivatives Clearing Protocol**: this is a decentralised, non-custodial clearing protocol that provides the financial logic for calculating margin requirements, facilitating liquidations and enabling P\&L attribution. The margin system within the protocol is advanced, enabling cross-collateralisation of tokens different to the settlement token, along with full portfolio cross-margining. This means trading and providing liquidity is highly capital efficient. For example, ETH-BTC perp trading can be over 350% more capital efficient than on other networks.
* **Stablecoin Protocol**: trades on Reya Network can settle in any governance-approved asset. However, the majority of markets will settle in rUSD, a stablecoin that is native to Reya Network. This allows for neutral settlement between users bridging from different chains. It will also enable the implementation of a payments clearing protocol to help facilitate settlement, as well as a savings protocol that will make it attractive to not just trade in rUSD, but to hold wealth in it, ensuring rUSD transcends from payments settlement into a long-term store of value.

### **A Liquidity Layer**

A Passive-LP Pool is embedded into the network. This leverages Reya Network’s ability to generate price indices to create contracts at an index. In the first instance external indices will be used, allowing traders to get exposure at a well defined index with reduced execution risk. The mechanism is built on the derivatives clearing logic, and therefore inherits the same features — making it highly capital efficient for passive LPs. This mechanism also acts as the ultimate liquidity backstop of the system and boosts the liquidity of any exchange operating on Reya Network.

### **Governance**

Reya Network will decentralise, ensuring the network isn’t controlled by any single entity and providing transparency and trust through to the end users of the system. Trust and transparency also improves the composability of the network giving other exchanges confidence they can operate on the network, too. In turn this will help accelerate adoption of Reya Network and further deepen liquidity for end users.
