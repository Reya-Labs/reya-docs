# Derivatives Clearing

## A clearing protocol built for capital efficiency

Reya Derivatives Clearing is a **non-custodial clearing protocol for derivatives instruments.** The Derivatives Clearing Protocol is the cornerstone that allows Reya to act as a self-reinforcing liquidity network. One of its distinguishing features is that it can aggregate positions from multiple markets _and_ multiple exchanges into a single margin account. In particular, this provides both traders and LPs with unprecedented capital efficiency by offsetting:

1. **Losses from one position with profits another**, and enabling leveraging of all portfolio profits immediately
2. **Margin requirements,** by recognizing the P\&L offset that can happen between positions
3. **P\&L and margin&#x20;**_**across**_**&#x20;exchanges,** by aggregating all positions into a single settlement and clearing layer

Connecting exchanges can be both DeFi or CeFi venues supporting the trading of any linear derivative contract (e.g., Futures, Swaps) via any on- or off-chain trading mechanism (e.g. Orderbook, AMM, Passive LP pool). By deploying its own appchain, Reya aims at providing these benefits while bringing on-chain the experience and speed of traditional centralized exchanges.

The basic concept in the Derivatives Clearing Protocol is that of a _collateral pool._ This is a compartment within the smart contract where margin deposits for any given market are kept and managed. Each of the user’s margin accounts is associated with one such collateral pool. Cashflow obligations are settled peer-to-pool. To make sure these are properly settled, the protocol has three main components:

1. Margin system that manages margin requirements for different settlement tokens and ensures safe levels of collateralization
2. Cross-collateralization module that manages features around collateralization with tokens other than the settlement token, as well as the auto-exchange mechanism which forces conversion to manage risk levels
3. Liquidation module that manages liquidation procedures when an account is declared liquidateable by the margin system.

<figure><img src="../.gitbook/assets/image (30).png" alt=""><figcaption></figcaption></figure>

## Peer-to-pool, non-custodial, on-chain trading

Reya fully embraces the non-custodial nature of DeFi. This means that the entire flow of execution, clearing and settlement is done on the blockchain itself, in a transparent and secure way. Every user can verify their transactions as they get registered, as well as their balances, margin requirements and accrued cashflows.

The Derivatives Clearing Protocol is non-custodial, meaning that all settlement and margin funds are are safeguarded cryptographically by the Protocol rather than by an administrator (eliminating ‘FTX’ risk). The Protocol includes mechanisms to ensure that all funds are correctly managed and settled.

Furthermore, the architecture of the Derivatives Clearing around collateral pools means that trading in Reya is peer-to-pool: although positions are originated in pairs at associated exchanges to ensure pool neutrality, subsequent margining and settlement is between the user and the collateral pool itself. In particular, the counterparty at the origin of the position might completely unwind their exposure with no difference for the user.

Peer-to-pool trading is the decentralized version of the CCP’s novation: rather than splitting a contract into two, between each of the two counterparties and a central party, the smart contract and the collateral pool itself lies at the center of the system, with every party facing it. Importantly, the single insolvency and liquidation of the counterparty has no direct bearing on a user, and their settlement cashflows are guaranteed as long as the pool as a whole remains solvent. Accordingly, if the pool’s solvency is threatened by a contract or instrument, positions are shortcut or, in the extreme case, losses are distributed in a socialized manner through auto-deleveraging as discussed below.

## Margin accounts

Users can create multiple accounts per wallet, where each account is represented as an ERC-721 Token. This provides users with the ability to create segregated accounts for trades they don’t want cross-margined with other positions. The use of an ERC-721 Token also provides the following benefits:

* Users can transfer these accounts to another wallet without unwinding any positions. In many ways, this is akin to in-specie transfers and provides a significant improvement in functionality for sophisticated actors on the protocol.
* Segregated account owners, which can be EOAs or smart contracts, are able to prescribe granular permissioning rules to other EOA and smart contract addresses. This may, for example, allow a user to have a cold wallet for withdrawals but grant permissions to their hot wallet to conduct trades without giving it withdrawal access
* Since addresses are decoupled from accounts, it is possible to layer in further abstractions that govern delegation and authorization flows improving account management operational security.
