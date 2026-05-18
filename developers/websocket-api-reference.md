# WebSocket Info API Reference

## Overview

The Reya DEX Trading WebSocket API v2 provides real-time streaming data for decentralized exchange operations on the Reya Network. This version offers user-friendly data structures with human-readable formats, removing blockchain-specific details while maintaining comprehensive trading functionality.

For placing and cancelling orders over WebSocket, see [WebSocket Order Entry API Reference](ws-exec-api-reference.md). The recommended Market Maker integration runs both connections in parallel: that surface for order entry, this surface for read-side fanout.

## Server Endpoints

### Production Environment

* **URL**: `wss://ws.reya.xyz`
* **Protocol**: WSS
* **Description**: Production WebSocket server for live trading

### Staging Environment

* **URL**: `wss://websocket-staging.reya.xyz`
* **Protocol**: WSS
* **Description**: Staging WebSocket server for pre-production testing

### Test Environment

* **URL**: `wss://websocket-testnet.reya.xyz`
* **Protocol**: WSS
* **Description**: Test WebSocket server for development

## Channel Architecture

The API uses a hierarchical channel structure with clear separation between different data types:

{% stepper %}
{% step %}
### Market Data Channels

* `/v2/markets/summary` - Perp market summaries
* `/v2/market/{symbol}/summary` - Individual perp market summary
* `/v2/spotMarkets/summary` - Spot market summaries
* `/v2/spotMarket/{symbol}/summary` - Individual spot market summary
* `/v2/market/{symbol}/perpExecutions` - Market-specific perpetual executions
* `/v2/market/{symbol}/depth` - L2 order book depth snapshots, only relevant for markets using the Reya Order Book instead of the AMM
* `/v2/market/{symbol}/spotExecutions` - Market-specific spot executions
* `/v2/market/{symbol}/spotExecutionBusts` - Market-specific spot execution busts (failed spot fills)
* `/v2/prices` - All symbol prices
* `/v2/prices/{symbol}` - Individual symbol prices
{% endstep %}

{% step %}
### Wallet Data Channels

* `/v2/wallet/{address}/positions` - Position updates
* `/v2/wallet/{address}/orderChanges` - Order change updates
* `/v2/wallet/{address}/perpExecutions` - Wallet-specific perpetual executions
* `/v2/wallet/{address}/spotExecutions` - Wallet-specific spot executions
* `/v2/wallet/{address}/spotExecutionBusts` - Wallet-specific spot execution busts
* `/v2/wallet/{address}/accountBalances` - Account balance updates
{% endstep %}
{% endstepper %}

### Parameter Validation

#### Symbol Parameter

* **Pattern**: `^[A-Za-z0-9]+$`
* **Examples**: `BTCRUSDPERP`, `ETHRUSD`, `kBONKRUSDPERP`, `AI16ZRUSDPERP`
* **Description**: Trading symbol supporting alphanumeric characters

#### Address Parameter

* **Pattern**: `^0x[a-fA-F0-9]{40}$`
* **Example**: `0x6c51275fd01d5dbd2da194e92f920f8598306df2`
* **Description**: Ethereum wallet address (40 hexadecimal characters)

## Message Structure

The Info surface uses one envelope shape for streamed channel data, plus a small set of control envelopes for subscription management. All envelopes share a `type` discriminator at the top level; the rest of the body is type-specific.

### Channel Data Envelope (Server → Client)

```json
{
  "type": "channel_data",
  "timestamp": 1747927089946,
  "channel": "/v2/market/BTCRUSDPERP/summary",
  "data": { /* channel-specific data */ }
}
```

* **type**: Always `"channel_data"` for data updates
* **timestamp**: Server timestamp in milliseconds
* **channel**: Specific channel identifier
* **data**: Channel-specific payload (object or array)

### Subscribe Envelope (Client → Server)

```json
{
  "type": "subscribe",
  "channel": "/v2/markets/summary",
  "id": "req123"
}
```

The `id` is an optional client-chosen correlation marker. The server does not echo it back in the confirmation and does not enforce uniqueness across in-flight subscribes; it is purely for client-side bookkeeping.

### Subscribed Confirmation (Server → Client)

```json
{
  "type": "subscribed",
  "channel": "/v2/markets/summary",
  "contents": { /* optional initial data */ }
}
```

The `contents` field carries an initial snapshot for channels that provide one (e.g. `/v2/market/{symbol}/depth`); otherwise it is omitted.

### Unsubscribe Envelope (Client → Server)

```json
{
  "type": "unsubscribe",
  "channel": "/v2/markets/summary",
  "id": "req123"
}
```

### Unsubscribed Confirmation (Server → Client)

```json
{
  "type": "unsubscribed",
  "channel": "/v2/markets/summary"
}
```

### Error Envelope (Server → Client)

```json
{
  "type": "error",
  "message": "Invalid channel",
  "channel": "/v2/invalid/channel"
}
```

The shape and the full set of possible `message` values are documented in [Error Catalog](#error-catalog) below.

### Heartbeats

The heartbeat / connection-liveness mechanism is documented in detail on its own page — see [Heartbeats](heartbeats.md). Short version: protocol-level pings handle liveness automatically, no application-level code is required on the client.

## Channels Reference

### 1. Market Data Channels

#### `/v2/markets/summary`

**Purpose**: Real-time updates for all market summaries

**Subscription**:

```json
{
  "type": "subscribe",
  "channel": "/v2/markets/summary"
}
```

**Message Structure**:

```json
{
  "type": "channel_data",
  "timestamp": 1747927089946,
  "channel": "/v2/markets/summary",
  "data": [
    {
      "symbol": "BTCRUSDPERP",
      "updatedAt": 1747927089946,
      "longOiQty": "154.741",
      "shortOiQty": "154.706",
      "oiQty": "154.741",
      "fundingRate": "-0.000509373441021089",
      "longFundingValue": "412142.26",
      "shortFundingValue": "412142.26",
      "fundingRateVelocity": "-0.00000006243",
      "volume24h": "917833.49891",
      "pxChange24h": "92.6272285500004",
      "throttledOraclePrice": "2666.48162040777",
      "throttledPoolPrice": "2666.48166680625",
      "pricesUpdatedAt": 1747927089597
    }
  ]
}
```

<details>

<summary><strong>Data Type - MarketSummary</strong></summary>

* `symbol` (string): Trading symbol
* `updatedAt` (integer): Last calculation timestamp (milliseconds)
* `longOiQty` (string): Long open interest in lots
* `shortOiQty` (string): Short open interest in lots
* `oiQty` (string): Total open interest quantity
* `fundingRate` (string): Current hourly funding rate
* `longFundingValue` (string): Current long funding value
* `shortFundingValue` (string): Current short funding value
* `fundingRateVelocity` (string): Funding rate velocity
* `volume24h` (string): 24-hour trading volume
* `pxChange24h` (string, optional): 24-hour price change
* `throttledOraclePrice` (string, optional): Last oracle price at summary update
* `throttledPoolPrice` (string, optional): Last pool price at summary update
* `pricesUpdatedAt` (integer, optional): Last price update timestamp

</details>

#### `/v2/market/{symbol}/summary`

**Purpose**: Real-time updates for a specific market's summary

**Parameters**:

* `symbol`: Trading symbol (e.g., `BTCRUSDPERP`, `kBONKRUSDPERP`)

**Subscription**:

```json
{
  "type": "subscribe",
  "channel": "/v2/market/BTCRUSDPERP/summary"
}
```

**Message Structure**:

```json
{
  "type": "channel_data",
  "timestamp": 1747927089946,
  "channel": "/v2/market/BTCRUSDPERP/summary",
  "data": {
    "symbol": "BTCRUSDPERP",
    "updatedAt": 1747927089946,
    "longOiQty": "154.741",
    "shortOiQty": "154.706",
    "oiQty": "154.741",
    "fundingRate": "-0.000509373441021089",
    "longFundingValue": "412142.26",
    "shortFundingValue": "412142.26",
    "fundingRateVelocity": "-0.00000006243",
    "volume24h": "917833.49891",
    "pxChange24h": "92.6272285500004",
    "throttledOraclePrice": "2666.48162040777",
    "throttledPoolPrice": "2666.48166680625",
    "pricesUpdatedAt": 1747927089597
  }
}
```

<details>

<summary><strong>Data Type - MarketSummary</strong></summary>

Same as above - see `/v2/markets/summary` channel for complete field definitions.

</details>

#### `/v2/spotMarkets/summary`

**Purpose**: Real-time updates for all spot market summaries

**Subscription**:

```json
{
  "type": "subscribe",
  "channel": "/v2/spotMarkets/summary"
}
```

**Message Structure**:

```json
{
  "type": "channel_data",
  "timestamp": 1747927089946,
  "channel": "/v2/spotMarkets/summary",
  "data": [
    {
      "symbol": "WETHRUSD",
      "updatedAt": 1747927089946,
      "volume24h": "917833.49891",
      "pxChange24h": "92.6272285500004",
      "oraclePrice": "2666.48162040777"
    }
  ]
}
```

<details>

<summary><strong>Data Type - SpotMarketSummary</strong></summary>

* `symbol` (string): Trading symbol
* `updatedAt` (integer): Last calculation timestamp (milliseconds)
* `volume24h` (string): 24-hour trading volume in USD
* `pxChange24h` (string, optional): Absolute 24-hour price change
* `oraclePrice` (string, optional): Current oracle price

</details>

#### `/v2/spotMarket/{symbol}/summary`

**Purpose**: Real-time updates for a specific spot market's summary

**Parameters**:

* `symbol`: Trading symbol (e.g., `WETHRUSD`)

**Subscription**:

```json
{
  "type": "subscribe",
  "channel": "/v2/spotMarket/WETHRUSD/summary"
}
```

**Message Structure**:

```json
{
  "type": "channel_data",
  "timestamp": 1747927089946,
  "channel": "/v2/spotMarket/WETHRUSD/summary",
  "data": {
    "symbol": "WETHRUSD",
    "updatedAt": 1747927089946,
    "volume24h": "917833.49891",
    "pxChange24h": "92.6272285500004",
    "oraclePrice": "2666.48162040777"
  }
}
```

<details>

<summary><strong>Data Type - SpotMarketSummary</strong></summary>

Same as above - see `/v2/spotMarkets/summary` channel for complete field definitions.

</details>

#### `/v2/market/{symbol}/perpExecutions`

**Purpose**: Real-time perpetual executions for a specific market

**Parameters**:

* `symbol`: Trading symbol (e.g., `BTCRUSDPERP`, `AI16ZRUSDPERP`)

**Subscription**:

```json
{
  "type": "subscribe",
  "channel": "/v2/market/BTCRUSDPERP/perpExecutions"
}
```

**Message Structure**:

```json
{
  "type": "channel_data",
  "timestamp": 1747927089946,
  "channel": "/v2/market/BTCRUSDPERP/perpExecutions",
  "data": [
    {
      "exchangeId": 1,
      "symbol": "BTCRUSDPERP",
      "accountId": 12345,
      "qty": "1.0",
      "side": "B",
      "price": "43000.00",
      "fee": "0.50",
      "type": "ORDER_MATCH",
      "timestamp": 1747927089946,
      "sequenceNumber": 152954
    }
  ]
}
```

<details>

<summary><strong>Data Type - PerpExecution</strong></summary>

* `exchangeId` (integer): Exchange identifier
* `symbol` (string): Trading symbol
* `accountId` (integer): Account identifier
* `qty` (string): Execution quantity
* `side` (Side): Execution side (B=Buy, A=Sell)
* `fee` (string): Total execution fee in rUSD
* `openingFee` (string, optional): Opening fee portion of the total fee in rUSD. Absent for position-extending executions.
* `price` (string): Execution price
* `type` (ExecutionType): Execution type (ORDER\_MATCH, LIQUIDATION, ADL)
* `timestamp` (integer): Execution timestamp (milliseconds)
* `sequenceNumber` (integer): Global sequence number
* `realizedPnl` (string, optional): Realized PnL from this execution in rUSD (priceVariationPnl + fundingPnl). Absent for position-extending executions.
* `priceVariationPnl` (string, optional): PnL component from price movement in rUSD. Absent for position-extending executions.
* `fundingPnl` (string, optional): PnL component from funding payments in rUSD. Absent for position-extending executions.

</details>

#### `/v2/prices`

**Purpose**: Real-time price updates for all symbols

**Subscription**:

```json
{
  "type": "subscribe",
  "channel": "/v2/prices"
}
```

**Message Structure**:

```json
{
  "type": "channel_data",
  "timestamp": 1747927089946,
  "channel": "/v2/prices",
  "data": [
    {
      "symbol": "BTCRUSDPERP",
      "oraclePrice": "43000.00",
      "poolPrice": "42999.50",
      "updatedAt": 1747927089946
    },
    {
      "symbol": "ETHRUSDPERP",
      "oraclePrice": "2500.00",
      "poolPrice": "2499.75",
      "updatedAt": 1747927089946
    }
  ]
}
```

<details>

<summary><strong>Data Type - Price</strong></summary>

* `symbol` (string): Trading symbol
* `oraclePrice` (string): Oracle price - Price given by the Stork feeds, used both as the peg price for prices on Reya, as well as Mark Prices
* `poolPrice` (string, optional): Pool price - The price currently quoted by the AMM for zero volume
* `updatedAt` (integer): Last update timestamp (milliseconds)

</details>

#### `/v2/prices/{symbol}`

**Purpose**: Real-time price updates for a specific symbol

**Parameters**:

* `symbol`: Trading symbol (e.g., `BTCRUSDPERP`, `kBONKRUSDPERP`)

**Subscription**:

```json
{
  "type": "subscribe",
  "channel": "/v2/prices/BTCRUSDPERP"
}
```

**Message Structure**:

```json
{
  "type": "channel_data",
  "timestamp": 1747927089946,
  "channel": "/v2/prices/BTCRUSDPERP",
  "data": {
    "symbol": "BTCRUSDPERP",
    "oraclePrice": "43000.00",
    "poolPrice": "42999.50",
    "updatedAt": 1747927089946
  }
}
```

<details>

<summary><strong>Data Type - Price</strong></summary>

Same as above - see `/v2/prices` channel for complete field definitions.

</details>

#### `/v2/market/{symbol}/depth`

**Purpose**: Real-time L2 order book depth snapshots for a specific market

**Parameters**:

* `symbol`: Trading symbol (e.g., `BTCRUSDPERP`, `DOGERUSDPERP`)

**Subscription**:

```json
{
  "type": "subscribe",
  "channel": "/v2/market/BTCRUSDPERP/depth"
}
```

**Message Structure**:

```json
{
  "type": "channel_data",
  "timestamp": 1747927089946,
  "channel": "/v2/market/BTCRUSDPERP/depth",
  "data": {
    "symbol": "BTCRUSDPERP",
    "type": "SNAPSHOT",
    "bids": [
      { "px": "42999.50", "qty": "1.5" },
      { "px": "42998.00", "qty": "2.0" }
    ],
    "asks": [
      { "px": "43000.50", "qty": "1.0" },
      { "px": "43001.00", "qty": "3.0" }
    ],
    "updatedAt": 1747927089946
  }
}
```

<details>

<summary><strong>Data Type - Depth</strong></summary>

* `symbol` (string): Trading symbol
* `type` (DepthType): Depth message type (SNAPSHOT, UPDATE)
* `bids` (array): Bid side levels aggregated by price, sorted descending by price
  * `px` (string): Price level
  * `qty` (string): Aggregated quantity at this price level
* `asks` (array): Ask side levels aggregated by price, sorted ascending by price
  * `px` (string): Price level
  * `qty` (string): Aggregated quantity at this price level
* `updatedAt` (integer): Snapshot generation timestamp (milliseconds)

</details>

#### `/v2/market/{symbol}/spotExecutions`

**Purpose**: Real-time spot executions for a specific market

**Parameters**:

* `symbol`: Trading symbol (e.g., `ETHRUSD`, `BTCRUSD`)

**Subscription**:

```json
{
  "type": "subscribe",
  "channel": "/v2/market/ETHRUSD/spotExecutions"
}
```

**Message Structure**:

```json
{
  "type": "channel_data",
  "timestamp": 1747927089946,
  "channel": "/v2/market/ETHRUSD/spotExecutions",
  "data": [
    {
      "exchangeId": 1,
      "symbol": "ETHRUSD",
      "accountId": 12345,
      "makerAccountId": 67890,
      "orderId": "63552420354981888",
      "makerOrderId": "63552420037263360",
      "qty": "1.0",
      "side": "B",
      "price": "2500.00",
      "fee": "0.0",
      "type": "ORDER_MATCH",
      "timestamp": 1747927089946,
      "sequenceNumber": 152954
    }
  ]
}
```

<details>

<summary><strong>Data Type - SpotExecution</strong></summary>

* `exchangeId` (integer, optional): Exchange identifier
* `symbol` (string): Trading symbol
* `accountId` (integer): Account identifier of the taker side of the trade
* `makerAccountId` (integer): Maker account ID (counterparty providing liquidity)
* `orderId` (string, optional): Taker-side order ID. Absent when the taker order was filled and removed in the same matching round.
* `makerOrderId` (string, optional): Maker-side order ID. Absent when the maker order was fully filled in this execution.
* `qty` (string): Execution quantity in base asset units
* `side` (Side): Taker side (B=Buy, A=Sell). The maker is always the opposite side.
* `price` (string): Execution price in quote-per-base units
* `fee` (string): Fee charged to the taker, in the market's fee asset
* `type` (ExecutionType): Execution type (ORDER\_MATCH, LIQUIDATION, ADL)
* `timestamp` (integer): Execution timestamp (milliseconds since epoch)
* `sequenceNumber` (integer): Monotonic per-execution sequence number across the spot matching engine; increases by 1 for every spot execution on Reya. Use this to dedup and gap-detect on the consumer side after a reconnect.

</details>

#### `/v2/market/{symbol}/spotExecutionBusts`

**Purpose**: Real-time spot execution busts (failed spot fills) for a specific market

**Parameters**:

* `symbol`: Trading symbol (e.g., `ETHRUSD`, `BTCRUSD`)

**Subscription**:

```json
{
  "type": "subscribe",
  "channel": "/v2/market/ETHRUSD/spotExecutionBusts"
}
```

**Message Structure**:

```json
{
  "type": "channel_data",
  "timestamp": 1747927089946,
  "channel": "/v2/market/ETHRUSD/spotExecutionBusts",
  "data": [
    {
      "symbol": "ETHRUSD",
      "accountId": 12345,
      "exchangeId": 1,
      "makerAccountId": 67890,
      "orderId": "63552420354981888",
      "makerOrderId": "63552420037263360",
      "qty": "1.0",
      "side": "B",
      "price": "2500.00",
      "reason": "08c379a0...",
      "timestamp": 1747927089946
    }
  ]
}
```

<details>

<summary><strong>Data Type - SpotExecutionBust</strong></summary>

A bust is emitted when the matching engine matched two orders but the on-chain settlement attempt reverted (e.g. insufficient balance, signature staleness, market paused). The match is rolled back; both orders are released back to their owners' state. See [Trade Busts](trade-busts.md) for the full trade lifecycle, when busts happen, and how clients should handle them.

* `symbol` (string): Trading symbol
* `accountId` (integer): Account identifier of the taker side of the failed trade
* `exchangeId` (integer): Exchange identifier
* `makerAccountId` (integer): Maker account ID (counterparty)
* `orderId` (string): Taker-side order ID
* `makerOrderId` (string): Maker-side order ID
* `qty` (string): Failed base quantity in base asset units
* `side` (Side): Taker side (B=Buy, A=Sell)
* `price` (string): Price at which the failed match was attempted
* `reason` (string): Hex-encoded revert reason bytes from the on-chain settlement attempt. Clients can ABI-decode this against the OrdersGateway error ABI to recover the specific revert (e.g. `InsufficientBalance`, `UnauthorizedSigner`). The first 4 bytes are the selector; subsequent bytes are the ABI-encoded args.
* `timestamp` (integer): Block timestamp of the failed settlement (milliseconds since epoch). This is the chain-side timestamp, not the original off-chain match timestamp.

</details>

### 2. Wallet Data Channels

#### `/v2/wallet/{address}/positions`

**Purpose**: Real-time position updates for a wallet

**Parameters**:

* `address`: Ethereum wallet address (e.g., `0x6c51275fd01d5dbd2da194e92f920f8598306df2`)

**Subscription**:

```json
{
  "type": "subscribe",
  "channel": "/v2/wallet/0x6c51275fd01d5dbd2da194e92f920f8598306df2/positions"
}
```

**Message Structure**:

```json
{
  "type": "channel_data",
  "timestamp": 1747927089946,
  "channel": "/v2/wallet/0x6c51275fd01d5dbd2da194e92f920f8598306df2/positions",
  "data": [
    {
      "exchangeId": 1,
      "symbol": "BTCRUSDPERP",
      "accountId": 12345,
      "qty": "1.5",
      "side": "B",
      "avgEntryPrice": "43000.00",
      "avgEntryFundingValue": "100.25",
      "lastTradeSequenceNumber": 152954
    }
  ]
}
```

<details>

<summary><strong>Data Type - Position</strong></summary>

* `exchangeId` (integer): Exchange identifier
* `symbol` (string): Trading symbol
* `accountId` (integer): Account identifier
* `qty` (string): Position quantity
* `side` (Side): Position side (B=Buy, A=Sell)
* `avgEntryPrice` (string): Average entry price
* `avgEntryFundingValue` (string): Average entry funding value
* `lastTradeSequenceNumber` (integer): Last execution sequence number

</details>

#### `/v2/wallet/{address}/orderChanges`

**Purpose**: Real-time order change updates for wallet

**Parameters**:

* `address`: Ethereum wallet address

**Subscription**:

```json
{
  "type": "subscribe",
  "channel": "/v2/wallet/0x6c51275fd01d5dbd2da194e92f920f8598306df2/orderChanges"
}
```

**Message Structure**:

```json
{
  "type": "channel_data",
  "timestamp": 1747927089946,
  "channel": "/v2/wallet/0x6c51275fd01d5dbd2da194e92f920f8598306df2/orderChanges",
  "data": [
    {
      "exchangeId": 1,
      "symbol": "BTCRUSDPERP",
      "accountId": 12345,
      "orderId": "123456789-123123123",
      "qty": "1.0",
      "execQty": "0.5",
      "side": "B",
      "limitPx": "43000.00",
      "orderType": "LIMIT",
      "triggerPx": "50000.0",
      "timeInForce": "GTC",
      "reduceOnly": false,
      "status": "OPEN",
      "createdAt": 1747927089946,
      "lastUpdateAt": 1747927089946
    }
  ]
}
```

<details>

<summary><strong>Data Type - Order</strong></summary>

* `exchangeId` (integer): Exchange identifier
* `symbol` (string): Trading symbol
* `accountId` (integer): Account identifier
* `side` (Side): Order side (B=Buy, A=Sell)
* `limitPx` (string): Limit price
* `orderType` (OrderType): Order type (LIMIT, TP, SL)
* `status` (OrderStatus): Order status (OPEN, FILLED, CANCELLED, REJECTED)
* `createdAt` (integer): Creation timestamp (milliseconds)
* `lastUpdateAt` (integer): Last update timestamp (milliseconds)
* `orderId` (string): Order identifier
* `qty` (string, optional): Order quantity
* `execQty` (string, optional): Executed quantity in the current order update
* `cumQty` (string, optional): Total executed quantity across all fills
* `triggerPx` (string, optional): Trigger price for TP/SL orders
* `timeInForce` (TimeInForce, optional): Time in force (IOC, GTC)
* `reduceOnly` (boolean, optional): Reduce-only flag (exclusively for LIMIT IOC orders)

</details>

#### `/v2/wallet/{address}/perpExecutions`

**Purpose**: Real-time perpetual execution updates for a wallet

**Parameters**:

* `address`: Ethereum wallet address

**Subscription**:

```json
{
  "type": "subscribe",
  "channel": "/v2/wallet/0x6c51275fd01d5dbd2da194e92f920f8598306df2/perpExecutions"
}
```

**Message Structure**:

```json
{
  "type": "channel_data",
  "timestamp": 1747927089946,
  "channel": "/v2/wallet/0x6c51275fd01d5dbd2da194e92f920f8598306df2/perpExecutions",
  "data": [
    {
      "exchangeId": 1,
      "symbol": "BTCRUSDPERP",
      "accountId": 12345,
      "qty": "1.0",
      "side": "B",
      "price": "43000.00",
      "fee": "0.50",
      "type": "ORDER_MATCH",
      "timestamp": 1747927089946,
      "sequenceNumber": 152954
    }
  ]
}
```

<details>

<summary><strong>Data Type - PerpExecution</strong></summary>

Same as above - see `/v2/market/{symbol}/perpExecutions` channel for complete field definitions.

</details>

#### `/v2/wallet/{address}/spotExecutions`

**Purpose**: Real-time spot execution updates for a wallet

**Parameters**:

* `address`: Ethereum wallet address

**Subscription**:

```json
{
  "type": "subscribe",
  "channel": "/v2/wallet/0x6c51275fd01d5dbd2da194e92f920f8598306df2/spotExecutions"
}
```

**Message Structure**:

```json
{
  "type": "channel_data",
  "timestamp": 1747927089946,
  "channel": "/v2/wallet/0x6c51275fd01d5dbd2da194e92f920f8598306df2/spotExecutions",
  "data": [
    {
      "exchangeId": 1,
      "symbol": "ETHRUSD",
      "accountId": 12345,
      "makerAccountId": 67890,
      "orderId": "63552420354981888",
      "makerOrderId": "63552420037263360",
      "qty": "1.0",
      "side": "B",
      "price": "2500.00",
      "fee": "0.0",
      "type": "ORDER_MATCH",
      "timestamp": 1747927089946,
      "sequenceNumber": 152954
    }
  ]
}
```

<details>

<summary><strong>Data Type - SpotExecution</strong></summary>

Same as above - see `/v2/market/{symbol}/spotExecutions` channel for complete field definitions.

</details>

#### `/v2/wallet/{address}/spotExecutionBusts`

**Purpose**: Real-time spot execution bust updates for a wallet

**Parameters**:

* `address`: Ethereum wallet address

**Subscription**:

```json
{
  "type": "subscribe",
  "channel": "/v2/wallet/0x6c51275fd01d5dbd2da194e92f920f8598306df2/spotExecutionBusts"
}
```

**Message Structure**:

```json
{
  "type": "channel_data",
  "timestamp": 1747927089946,
  "channel": "/v2/wallet/0x6c51275fd01d5dbd2da194e92f920f8598306df2/spotExecutionBusts",
  "data": [
    {
      "symbol": "ETHRUSD",
      "accountId": 12345,
      "exchangeId": 1,
      "makerAccountId": 67890,
      "orderId": "63552420354981888",
      "makerOrderId": "63552420037263360",
      "qty": "1.0",
      "side": "B",
      "price": "2500.00",
      "reason": "08c379a0...",
      "timestamp": 1747927089946
    }
  ]
}
```

<details>

<summary><strong>Data Type - SpotExecutionBust</strong></summary>

Same as above - see `/v2/market/{symbol}/spotExecutionBusts` channel for complete field definitions.

</details>

#### `/v2/wallet/{address}/accountBalances`

**Purpose**: Real-time account balance updates for a wallet

**Parameters**:

* `address`: Ethereum wallet address

**Subscription**:

```json
{
  "type": "subscribe",
  "channel": "/v2/wallet/0x6c51275fd01d5dbd2da194e92f920f8598306df2/accountBalances"
}
```

**Message Structure**:

```json
{
  "type": "channel_data",
  "timestamp": 1747927089946,
  "channel": "/v2/wallet/0x6c51275fd01d5dbd2da194e92f920f8598306df2/accountBalances",
  "data": [
    {
      "accountId": 12345,
      "asset": "WSTETH",
      "realBalance": "1.25",
      "balanceDEPRECATED": "1.25"
    }
  ]
}
```

<details>

<summary><strong>Data Type - AccountBalance</strong></summary>

* `accountId` (integer): Account identifier
* `asset` (string): Asset symbol (e.g., WSTETH, RUSD)
* `realBalance` (string): Sum of account net deposits and realized PnL from closed positions
* `balanceDEPRECATED` (string): Sum of account net deposits only (deprecated, will be removed)

</details>

## Error Catalog

The server emits an `error` envelope when it cannot process a frame. The connection stays open; only the offending operation is rejected. Every error envelope shares this shape:

```json
{
  "type": "error",
  "message": "<human-readable description>",
  "channel": "<channel path, present when applicable>"
}
```

The `channel` field is included when the error relates to a specific channel (e.g. an invalid subscribe target). It is omitted for frame-level errors that aren't tied to a particular channel.

The full set of `message` strings emitted by the server:

| Message | When emitted | Client action |
|---|---|---|
| `Invalid JSON` | The frame body could not be parsed as JSON. | Fix the client serializer. |
| `Invalid type` | The frame's `type` field is not one of `subscribe`, `unsubscribe`, `ping`. (`pong` is server-only — clients don't send JSON pong frames.) | Verify the request `type`. |
| `Invalid channel name` | The subscribe / unsubscribe target does not match a known channel path or has malformed parameters (e.g. an invalid symbol or address). | Check the channel name against the [Channels Reference](#channels-reference) and the [Parameter Validation](#parameter-validation) rules. |
| `Error while fetching snapshot from {channel}` | The server failed to compute the initial snapshot for a freshly-subscribed channel (typically a transient backend issue). The subscription is rolled back; the client may retry. | Retry the subscribe after a short backoff. If the problem persists, contact support with the channel name and timestamp. |

## Data Types & Schemas

### Enumeration Types

<details>

<summary><strong>Side</strong> - Order/position side indicator</summary>

* `B`: Buy/Bid
* `A`: Ask/Sell

</details>

<details>

<summary><strong>ExecutionType</strong> - Type of execution that occurred</summary>

* `ORDER_MATCH`: Regular order matching
* `LIQUIDATION`: Liquidation execution
* `ADL`: Auto-deleveraging execution

</details>

<details>

<summary><strong>OrderStatus</strong> - Current status of an order</summary>

* `OPEN`: Order is active and can be filled
* `FILLED`: Order has been completely filled
* `CANCELLED`: Order has been cancelled
* `REJECTED`: Order was rejected

</details>

<details>

<summary><strong>OrderType</strong> - Type of order placed</summary>

* `LIMIT`: Limit order
* `TP`: Take profit order
* `SL`: Stop loss order

</details>

<details>

<summary><strong>TimeInForce</strong> - Order duration specification</summary>

* `IOC`: Immediate or Cancel
* `GTC`: Good Till Cancel

</details>

<details>

<summary><strong>DepthType</strong> - Order book depth message type</summary>

* `SNAPSHOT`: Full order book snapshot
* `UPDATE`: Single level change update

</details>

<details>

<summary><strong>AccountType</strong> - Account type classification</summary>

* `MAINPERP`: Main perpetual trading account
* `SUBPERP`: Sub perpetual trading account
* `SPOT`: Spot trading only account

</details>

## Connection Management

### Reconnection Pattern

The reconnection algorithm (exponential backoff with jitter) and the post-reconnect re-subscribe / REST-reconcile steps are documented in detail on the [Reconnection Pattern](reconnection-pattern.md) page. The algorithm is shared with the Order Entry WebSocket; surface-specific post-reconnect actions for both are covered there.

### Graceful Shutdown

The server's drain-and-close behavior on rolling deploys (10s drain, `/ready` returns `503`, idle connections closed first, `1001 SERVER_SHUTTING_DOWN` on close) is shared with the Order Entry WebSocket and documented in [Server-Side Graceful Shutdown](heartbeats.md#server-side-graceful-shutdown).

## Python SDK Example

Worked examples are included in the [Reya Python SDK](https://github.com/Reya-Labs/reya-python-sdk) under [`examples/websocket/`](https://github.com/Reya-Labs/reya-python-sdk/tree/main/examples/websocket). The directory is split by market type:

* [`examples/websocket/perps/market_monitoring.py`](https://github.com/Reya-Labs/reya-python-sdk/blob/main/examples/websocket/perps/market_monitoring.py) — subscribe to perp market summaries
* [`examples/websocket/perps/prices_monitoring.py`](https://github.com/Reya-Labs/reya-python-sdk/blob/main/examples/websocket/perps/prices_monitoring.py) — subscribe to the price stream
* [`examples/websocket/perps/wallet_monitoring.py`](https://github.com/Reya-Labs/reya-python-sdk/blob/main/examples/websocket/perps/wallet_monitoring.py) — subscribe to wallet-scoped channels (positions, order changes, executions, balances)
* [`examples/websocket/spot/spot_executions.py`](https://github.com/Reya-Labs/reya-python-sdk/blob/main/examples/websocket/spot/spot_executions.py) — subscribe to spot execution streams
* [`examples/websocket/spot/depth_market_maker.py`](https://github.com/Reya-Labs/reya-python-sdk/blob/main/examples/websocket/spot/depth_market_maker.py) — bootstrap state via REST, then drive a market-maker loop off of `depth`, `accountBalances`, `openOrders`, and `spotExecutions` updates

Run any of them with:

```bash
poetry shell
python -m examples.websocket.perps.market_monitoring  # for example
```

See each script's docstring for prerequisites (`.env` setup, funded test accounts on cronos).
