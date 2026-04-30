---
description: >-
  This contract manages perpetual future markets, pricing of market orders
  executed against the passive pool, matching as well as keeping track of
  account exposures and PnLs in available markets
---

# Passive Perp Instrument

#### getUpdatedPositionInfo <a href="#getupdatedpositioninfo" id="getupdatedpositioninfo"></a>

```
function getUpdatedPositionInfo(uint128 marketId, uint128 accountId) external view returns (PerpPosition memory);
```

<details>

<summary>PerpPosition</summary>

```
/**
 * @notice Structure containing information about a position
 */
struct PerpPosition {
    /// The net base of the position
    SD59x18 base;
    /// The realized pnl of the position
    SD59x18 realizedPnL;
    /// The last price data used to mark-to-market the position
    PriceData lastPriceData;
    /// The last trackers used to update the position's base and realized pnl
    FundingAndADLTrackers trackers;
}

```

</details>

For a given market and margin account id, it returns the latest position information. The ‘base’ has precision 18 but represents the market underlying exposure, positive for long positions and negative for short ones. The realised PnL is represented in the quote token of the market, it shows PnL available for withdraw. The price and timestamp correspond to the last observation of the oracle fr the market underlying token. The trackers are internal auditing parameters used to update the position.

#### getPoolMaxExposures <a href="#getpoolmaxexposures" id="getpoolmaxexposures"></a>

```
function getPoolMaxExposures(uint128 marketId) external view returns (UD60x18 maxShortExposure, UD60x18 maxLongExposure)
```

Returns the maximum absolute exposure the current market can support in both directions (exposure supported by the Passive Pool exchange). The actual maximum amount that can be traded in one direction is only a proportion of this, amount controlled by the maxExposureFactor parameter. These values are in base amounts, with precision 18.

#### getLatestFundingRate <a href="#getlatestfundingrate" id="getlatestfundingrate"></a>

```
function getLatestFundingRate(uint128 marketId) external view returns (SD59x18)
```

Returns the latest funding rate of a given market. The funding rate has precision 18 decimals and represents the rate that will be applied per day and unit of price to obtain the funding rate PnL

#### getLatestMTMData <a href="#getlatestmtmdata" id="getlatestmtmdata"></a>

```
function getLatestMTMData(uint128 marketId) external view returns (PriceData memory);
```

<details>

<summary>PriceData</summary>

```
struct PriceData {
    /// The price represented with 18 decimals precision
    UD60x18 price;
    /// The timestamp at which the price was retrieved.
    uint256 timestamp;
}
```

</details>

Returns the latest mark-to-market (MTM) data: the pegging price (futures price provided by oracles) at the last MTM and its registration timestamp. If the MTM window hap passed but a transaction was not yet submitted to record the MTM event, it will return the current oracle price.

#### getFundingVelocity <a href="#getfundingvelocity" id="getfundingvelocity"></a>

```
function getFundingVelocity(uint128 marketId) external view returns (SD59x18);
```

Returns the latest funding velocity for a given market. This represents the change per day in funding rate and is determined by the Pool’s current slippage and a velocity multiplier parameter. The funding rate is continuously adjusted by this velocity.

#### getOpenBaseInterest <a href="#getopenbaseinterest" id="getopenbaseinterest"></a>

```
function getOpenBaseInterest(uint128 marketId) external view returns (UD60x18);
```

Returns the current open interest with WAD precision of the given market.

#### getInstantaneousPoolPrice <a href="#getinstantaneouspoolprice" id="getinstantaneouspoolprice"></a>

```
function getInstantaneousPoolPrice(uint128 marketId) external view returns (UD60x18);
```

Returns the current price offered by the Passive Pool. This price is equal to the oracle price adjusted by the Pool’s slippage. The price deviation (pSlippage) can either be negative or positive depending on the direction of the pool's net imbalance in the given market, always giving the Pool a better price. This is the price a taker would be given if the traded base is close to zero.

#### getSimulatedPoolPrice <a href="#getsimulatedpoolprice" id="getsimulatedpoolprice"></a>

```
function getSimulatedPoolPrice(uint128 marketId, SD59x18 orderBase) external view returns (UD60x18);
```

Returns the price the Passive Pool would offer for a match order of the given size. Similar to the `getInstantaneousPoolPrice`, this is the oracle price adjusted by the Pool’s slippage but also taking to account its exposure after the trade. This can be used to check the execution price before a trade is made.
