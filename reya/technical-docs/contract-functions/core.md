---
description: >-
  The margin engine contract which imposes margin requirements and coordinates
  liquidations, manages margin accounts and their collateral holdings
---

# Core

#### createAccount <a href="#createaccount" id="createaccount"></a>

```
function createAccount(address accountOwner) external returns (uint128 accountId);
```

Creates a margin account for the given address, returns its id. An address can have multiple margin accounts in order to separate exposure and margin.

#### activateFirstMarketForAccount <a href="#activatefirstmarketforaccount" id="activatefirstmarketforaccount"></a>

```
function activateFirstMarketForAccount(uint128 accountId, uint128 marketId) external;
```

This function is used to register an account to a collateral pool. In order to trade or to be a liquidator, the account needs to be registered to a collateral pool. This is set implicitly for trading but not for liquidations.

#### getUsdNodeMarginInfo <a href="#getusdnodemargininfo" id="getusdnodemargininfo"></a>

```
function getUsdNodeMarginInfo(uint128 accountId) external view returns (MarginInfo memory);
```

<details>

<summary>MarginInfo</summary>

Copy

```
struct MarginInfo {
    /// The collateral token for which the information below is defined
    address collateral;
    /// These are all amounts that are available to contribute to cover margin requirements.
    int256 marginBalance;
    /// The real balance is the balance that is in ‘cash’, that is, actually held in the settlement
    /// collateral and not as value of an instrument which settles in that collateral
    int256 realBalance;
    /// Difference between margin balance and initial margin requirement
    int256 initialDelta;
    /// Difference between margin balance and maintenance margin requirement
    int256 maintenanceDelta;
    /// Difference between margin balance and liquidation margin requirement
    int256 liquidationDelta;
    /// Difference between margin balance and dutch margin requirement
    int256 dutchDelta;
    /// Difference between margin balance and adl margin requirement
    int256 adlDelta;
    /// Difference between margin balance and initial buffer margin requirement (for backstop lps)
    int256 initialBufferDelta;
    /// Information required to compute health of position in the context of adl liquidations
    uint256 liquidationMarginRequirement;
}
```

</details>

Get the margin requirements and collateral balance for the entire account denoted in USD terms with 18 decimals precision. This means the balance in different collaterals is aggregated and the current exchange rate is used (with a small risk haircut applied). The real balance represents the net balance (deposits and withdrawals) plus the realised PnL, while the margin balance also includes the unrealised PnL. The delta represents the distance until the respective risk/liquidation threshold is hit (represented in USD as well). The liquidation margin requirement is the threshold for liquidation. If the margin balance depreciates under this value, the account will be liquidated.

#### getNodeMarginInfo <a href="#getnodemargininfo" id="getnodemargininfo"></a>

```
function getNodeMarginInfo(uint128 accountId, address collateral) external view returns (MarginInfo memory);
```

This function returns the same values as the `getUsdNodeMarginInfo` when the collateral is rUSD.

#### getTokenMarginInfo <a href="#gettokenmargininfo" id="gettokenmargininfo"></a>

```
function getTokenMarginInfo(uint128 accountId, address collateral) external view returns (MarginInfo memory);
```

This function returns the margin information in the given token with its respective decimal precision. Passing rUSD as collateral would result in the same value as getNodeMarginInfo if the account does not have margin in other tokens (e.g. sUSDe). If the account has margin in other tokens, it will aggregate the net margin and return the same values as `getUsdNodeMarginInfo`. When passing sUSDe as collateral, it will return only net balances and zero deltas and margin requirements as this token is not a quote token and accounts cannot carry exposure in it.

#### getCollateralInfo <a href="#getcollateralinfo" id="getcollateralinfo"></a>

```
function getCollateralInfo(uint128 accountId, address collateral) external view returns (CollateralInfo memory);
```

<details>

<summary>CollateralInfo</summary>

Copy

```
struct CollateralInfo {
    /// The account's collateral balance
    int256 netDeposits;
    /// These are all amounts that are available to contribute to cover margin requirements.
    int256 marginBalance;
    /// The real balance is the balance that is in ‘cash’, that is, actually held in the settlement
    /// collateral and not as value of an instrument which settles in that collateral
    int256 realBalance;
}
```

</details>

Get the collateral information of an account. The net deposits track only the in-and-out movements of the collateral, including deposits, withdrawals, fees paid. The real balance is the net plus the realised PnL gained from the markets exposure. This represents the amount that can be withdrawn and is also used to determine the liquidation status of this account. The margin balance also includes the unrealised PnL, which is not permitted for withdraw.

#### getActiveMarketsPerQuoteCollateral <a href="#getactivemarketsperquotecollateral" id="getactivemarketsperquotecollateral"></a>

```
function getActiveMarketsPerQuoteCollateral(uint128 accountId, address quoteCollateral) external view returns (uint128[] memory);
```

Returns the list of market IDs where the account is exposed. Note, rUSD is the only quote used so far.

#### triggerAutoExchange <a href="#triggerautoexchange" id="triggerautoexchange"></a>

```
function triggerAutoExchange(TriggerAutoExchangeInput memory input) external returns (AutoExchangeAmounts memory);
```

<details>

<summary>TriggerAutoExchangeInput</summary>

Copy

```
struct TriggerAutoExchangeInput {
    /*
     * @dev accountId The id of the auto-exchanged account.
     * @dev liquidatorAccountId The id of the liquidator's account.
     * @dev amountToAutoExchangeQuote The max amount of quote collaterals the liquidator wants to provide
     * @dev collateral The address of the collateral collateral to be paid to the liquidator
     * @dev inCollateral The address of the quote collateral the liquidator provides
    */
    uint128 accountId;
    uint128 liquidatorAccountId;
    uint256 requestedQuoteAmount;
    address collateral;
    address inCollateral;
}
```

</details>

Auto-exchange can be triggered if the user does not have enough quote tokens to cover exposure but owns collateral in other tokens. More details about the auto-exchange mechanics are found [here](https://docs.reya.network/derivatives-clearing/cross-collateralization/auto-exchange-conditions). If conditions apply, this function triggers an auto-exchange between the liquidator's quote and the account's collateral. The liquidator receives a discount on the collateral received based on the maximum between the amount requested by the liquidator, the maximum quote to be covered and the available collateral in the liquidated account.

The requestedQuoteAmount is denominated in rUSD, with 6 decimals precision. The `collateral` represents the token to be extracted by the liquidator and `inCollateral` represents the quote token, rUSD.

The following function can be used to calculate the amount that can be auto-exchanged: `calculateMaxQuoteToCoverInAutoExchange` and `calculateAvailableCollateralToBeAutoExchanged`.

<details>

<summary>AutoExchangeAmounts</summary>

Copy

```
struct AutoExchangeAmounts {
    /*
    * @dev collateralAmountToLiquidator Amount of collateral collaterals received by liquidator
    * @dev quoteAmountToIF Amount of quote collaterals paid to the insurance fund by the liquidator
    * @dev quoteAmountToAccount Amount of quote collaterals received by the auto-exchanged account
    */
    uint256 collateralAmountToLiquidator;
    uint256 quoteAmountToIF;
    uint256 quoteAmountToAccount;
}
```

</details>

This function returns AutoExchangeAmounts specifying the collateral tokens that went to the liquidator (denoted in the token with its own decimal precision), the quote amount that went to the insurance fund (small fee paid for the protocol’s safety) and the quote that reached the liquidated account.

#### calculateMaxQuoteToCoverInAutoExchange <a href="#calculatemaxquotetocoverinautoexchange" id="calculatemaxquotetocoverinautoexchange"></a>

```
function calculateMaxQuoteToCoverInAutoExchange(uint128 accountId, address inCollateral) external view returns (uint256);
```

Calculates the maximum amount of quote that can be covered by auto-exchange. The inCollateral should be rUSD and the output will be denoted in rUSD with 6 decimals precision. If the account id does not fall under any of the auto-exchange conditions [here](https://docs.reya.network/derivatives-clearing/cross-collateralization/auto-exchange-conditions)., this value is zero.

#### calculateAvailableCollateralToBeAutoExchanged <a href="#calculateavailablecollateraltobeautoexchanged" id="calculateavailablecollateraltobeautoexchanged"></a>

```
function calculateAvailableCollateralToBeAutoExchanged(uint128 accountId, address outCollateral, address quoteCollateral) external view returns (uint256);
```

Returns the maximum amount of collateral that can be extracted from the user’s account in exchange for quote. The out collateral represents the token to be received by the auto-exchanger and the quote collateral should be rUSD, the token paid by the auto-exchanger. It returns the amount denoted in the collateral token with its decimal precision.

#### execute <a href="#execute" id="execute"></a>

```
function execute(uint128 accountId, Command[] calldata commands) external returns (bytes[] memory outputs, MarginInfo memory marginInfo);
```

<details>

<summary>Command and CommandType</summary>

```
enum CommandType {
    Deposit, // (core command) deposit collaterals
    Withdraw, // (core command) withdraw collaterals
    DutchLiquidation, // (core command) dutch liquidation of an account
    MatchOrder, // (market command) propagation of matched orders
    TransferBetweenMarginAccounts // (core command) transfer between two margin accounts

}

struct Command {
    /**
     * @dev Identifies the command to be executed
     */
    CommandType commandType;
    /**
     * @dev Command inputs encoded in bytes
     */
    bytes inputs;
    /**
     * @dev Market id that identifies the instrument to execute
     * this command. If zero, the command will be sent to core.
     */
    uint128 marketId;
    /**
     * @dev Exchange id that identifies the exchange that executes
     * this command. If zero, the command does not involve an exchange (e.g. propagate cashflow)
     */
    uint128 exchangeId;
}
```

</details>

This is a router-type function, it allows the user to call multiple protocol functionalities in the same transaction e.g. deposit, trade, withdraw. For Core commands (Deposit, Withdraw, DutchLiquidation, TransferBetweenMarginAccounts) the market id and exchange id should be zero. For the MatchOrder, the values should be specified. These commands have to act on a single account account, the one globally specified in the function parameters. If the msg.sender does not have permissions to modify this account, the transaction will fail.

Each command requires a different set of encoded inputs, sent under bytes inputs:

* Deposit and Withdraw: `(address collateral, uint256 collateralAmount)`
* DutchLiquidation: (DutchLiquidationInput inputs) with `DutchLiquidationInput{uint128 liquidatableAccountId; address quoteCollateral; uint128[] marketIds; bytes[] inputs;}` where the inputs are encoded `(int56 liquidatedBase, uint256 priceLimit)` into bytes. This command allows the execution of Dutch liquidation on multiple markets in the same transaction.
* TransferBetweenMarginAccounts: (TransferInput inputs) with `TransferInput{uint128 destAccountId;address collateral;uint256 collateralAmount;}`
* MatchOrder: `(uint128[] memory counterpartyAccountIds, bytes memory orderInputs)`

`DutchLiquidation command`

This command triggers the liquidation of the `liquidatableAccountId` if its margin balance fell under the liquidation margin requirement, queried using `getUsdNodeMarginInfo` . This liquidation type moved the exposure (the `liquidatedBase`) to the liquidator at the market price (without the Pool slippage) within the specified price limit. The liquidator must remain above the initial margin requirements after this exchange, otherwise the transaction fails. The liquidator reward is a fixed percentage of the LMR delta, the improvement in the account's margin requirement after reducing exposure.

#### executeBySig <a href="#executebysig" id="executebysig"></a>

```
function executeBySig(uint128 accountId, Command[] calldata commands, EIP712Signature memory sig, bytes memory extraSignatureData) external returns (bytes[] memory outputs, MarginInfo memory marginInfo);
```

This function provides the same functionality as `execute()` with the added benefit that the msg.sender does not have to be the owner/admin of the provided `accountId`. The owner/admin’s signature is required but the transaction can be called from any address, enabling the use of relayer or external peripheries.

The EIP712Signature contains of the commands and inputs the owner intended to run, alongside a nonce, deadline. Read more about the EIP712 standard [here](https://eips.ethereum.org/EIPS/eip-712). The extraSignatureData is only a utility variable, can be left empty for external use.

#### executeBackstopLiquidation <a href="#executebackstopliquidation" id="executebackstopliquidation"></a>

```
function executeBackstopLiquidation(uint128 liquidatableAccountId, uint128 keeperAccountId, address quoteCollateral, UD60x18 backstopPercentage) external;
```

Triggers backstop liquidation of the gives account id. This sends the position to the backstop keeper (more details [here](https://docs.reya.network/derivatives-clearing/the-liquidation-engine/backstop-lps)). The backstop percentage is the percentage of the account's exposure to be transferred to the backstop LP.

Backstop is a type of liquidation that can be triggered on an account if its margin decreases below the ADL requirement. This can be queried using `getUsdNodeMarginInfo`. This type of liquidation sells the position at a better price to the designated backstop LP. The backstop LP account must be registered in Core. The keeper receives a set percentage of the liquidated amount (`backstopPercentage * liquidationMarginRequirement`).
