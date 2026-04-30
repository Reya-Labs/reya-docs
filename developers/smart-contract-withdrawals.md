# Smart Contract Withdrawals

As shown in the dApp funds on Reya Network are non-custodial and available to withdraw at anytime. You may use the withdrawal buttons on the dApp, or alternative you can withdraw directly from the Reya Network smart contracts.

**Follow the guide below to withdraw from the smart contracts.**

***

1.  Bridge ETH to Reya Network for gas fees and Socket bridge fees by using this bridge: [https://bridge.gelato.network/bridge/reya-network](https://bridge.gelato.network/bridge/reya-network). Note that:

    1. The bridge does not support ETH withdrawals for the time being, so make sure you don’t bridge too much ETH.
    2. Trading on Reya Network is made gas-free via a relayer architecture, if you’re interacting with the contracts directly you will need to pay a small amount of gas (hence the ETH needed).

    ![](https://docs.reya.network/~gitbook/image?url=https%3A%2F%2F4172979140-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FIHVxf7CaLQzjdZ5a8tyE%252Fuploads%252FaYn82zYOMm7iFAfnjhYp%252Fimage.png%3Falt%3Dmedia%26token%3D175dfb8f-b628-418b-8ed0-6874fd01cdbd\&width=768\&dpr=4\&quality=100\&sign=2bfc19e9\&sv=2)
2.  Withdraw funds from the Passive Pool into your wallet on Reya Network by calling the removeLiquidity function here: [https://usecannon.com/packages/reya-omnibus/latest/1729-main/interact/reya-omnibus/PassivePoolProxy/0xB4B77d6180cc14472A9a7BDFF01cc2459368D413#selector-0x0b7c92f9](https://usecannon.com/packages/reya-omnibus/latest/1729-main/interact/reya-omnibus/PassivePoolProxy/0xB4B77d6180cc14472A9a7BDFF01cc2459368D413#selector-0x0b7c92f9). To do this, you need to input the following parameters:

    1. poolId: 1
    2. sharesAmount: the amount you want to withdraw multiplied by 10^30 (10 to the power of 30).
    3. minAmount: the amount you want to withdraw multiplied by 10^6

    For example, if you want to withdraw 99.5 rUSD, the parameters will be:

    1. poolId: 1
    2. sharesAmount: 99500000000000000000000000000000
    3. minAmount: 99500000
3. Unwrap rUSD into USDC by calling the withdraw function here: [https://usecannon.com/packages/reya-omnibus/latest/1729-main/interact/reya-omnibus/RUSDProxy/0xa9F32a851B1800742e47725DA54a09A7Ef2556A3#selector-0x2e1a7d4d](https://usecannon.com/packages/reya-omnibus/latest/1729-main/interact/reya-omnibus/RUSDProxy/0xa9F32a851B1800742e47725DA54a09A7Ef2556A3#selector-0x2e1a7d4d). To do this, you need to input the following parameters:
   1. amount: the amount withdrawn at the previous step, same as minAmount (multiplied by 1000000). For example, if you withdrawn 99.5 rUSD, the parameters will be amount = 99500000
4. Confirm you have USDC in your Reya Wallet.
   1. Add the chain to your wallet. There is a link in the footer of: [https://explorer.reya.network/](https://explorer.reya.network/) ![](https://docs.reya.network/~gitbook/image?url=https%3A%2F%2F4172979140-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FIHVxf7CaLQzjdZ5a8tyE%252Fuploads%252FmQFS3KfuEx5eVQamrkW2%252Fimage.png%3Falt%3Dmedia%26token%3Db0825ced-080e-4ee0-9a0d-d9c8df6e03b3\&width=300\&dpr=4\&quality=100\&sign=aa83514e\&sv=2)
   2. Add the token at address: 0x3B860c0b53f2e8bd5264AA7c3451d41263C933F2 For example using Metamask: ![](https://docs.reya.network/~gitbook/image?url=https%3A%2F%2F4172979140-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FIHVxf7CaLQzjdZ5a8tyE%252Fuploads%252FlagVs3y8tJKlcI7VKyfG%252Fimage.png%3Falt%3Dmedia%26token%3D6513882f-6622-462f-b389-334a9149f93d\&width=300\&dpr=4\&quality=100\&sign=7f0b7758\&sv=2) Enter the contract address and the symbol and decimals should auto-populate. Press Next ![](https://docs.reya.network/~gitbook/image?url=https%3A%2F%2F4172979140-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FIHVxf7CaLQzjdZ5a8tyE%252Fuploads%252FDRHuHJdi2GeZp76DMDkc%252Fimage.png%3Falt%3Dmedia%26token%3D594bf99b-00ef-4112-8834-b278aa77be9e\&width=300\&dpr=4\&quality=100\&sign=d475d9df\&sv=2)
5.  Once confirmed, you should bridge funds from Reya Network to a source chain (Ethereum Mainnet/Arbitrum/Optimism/Polygon). To do this, you have to call the bridge function on the Socket contract here: [https://explorer.reya.network/address/0x1d43076909Ca139BFaC4EbB7194518bE3638fc76?tab=write\_contract#405e720a](https://explorer.reya.network/address/0x1d43076909Ca139BFaC4EbB7194518bE3638fc76?tab=write_contract#405e720a).

    ![](https://docs.reya.network/~gitbook/image?url=https%3A%2F%2F4172979140-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FIHVxf7CaLQzjdZ5a8tyE%252Fuploads%252FIImwsvtmdBvDQQD9BG05%252Fimage.png%3Falt%3Dmedia%26token%3Dfd994820-d3b2-42ba-abca-d2ff0b66b4d3\&width=300\&dpr=4\&quality=100\&sign=c66e47c8\&sv=2) To do this, you have to provide the following parameters:

    1. receiver: your wallet address on the source chain (the chain you are withdrawing to)
    2. amount: the amount withdrawn at the previous steps, same as minAmount (still multiplied by 1000000).
       1. note: the pull down at the end of the row will help you multiply by 10^6 but this number should be the same as the one used in the previous steps
    3. msgGasLimit: 10000000
    4.  connector: the socket connector address assigned to the source chain you want to withdraw to.

        NetworkConnector Address

        Ethereum Mainnet

        0x807B2e8724cDf346c87EEFF4E309bbFCb8681eC1

        Arbitrum

        0x663dc7E91157c58079f55C1BF5ee1BdB6401Ca7a

        Optimism

        0xe48AE3B68f0560d4aaA312E12fD687630C948561

        Polygon

        0x54CAA0946dA179425e1abB169C020004284d64D3
    5. execPayload: 0x
    6. options: 0x
    7. Send native ETH (uint256): the socket bridge fees. As these are dynamic, you can use 10000000000000000 to account for most cases (this is equivalent to 0.01 ETH). Note that you can use lower amount as well (e.g. 1000000000000000, which is equivalent to 0.001 ETH), but it might fail if Socket fees increase.

    Upon completion of these steps, funds should be in the destination wallet address entered, on the network corresponding to the network connecter address used, in 10-15min. If you have not received your funds after an hour, please open a support ticket in our Discord and we can try and help.
