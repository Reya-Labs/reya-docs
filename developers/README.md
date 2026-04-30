# Rest API Reference

**Base URL**: `https://api.reya.xyz/v2`

## Overview

The Reya DEX REST API v2 provides programmatic access to the Reya Perpetual Exchange, enabling traders to interact with the platform algorithmically. This API is designed for developers and professional traders who want to build automated trading systems, integrate with existing platforms, or develop custom interfaces for the Reya DEX ecosystem.

## Key Features

* **Market Data**: Access comprehensive market information, including asset definitions, market summaries, and real-time price data
* **Order Management**: Create, cancel, and monitor orders with support for various order types (limit, trigger)
* **Position Tracking**: Monitor your current positions and historical executions
* **Wallet Integration**: Manage wallet configurations and access wallet-specific data
* **Price Data**: Access historical price data with customizable time intervals through candle endpoints

## API Structure

The API is organized into several logical sections:

* **Reference Data**:  Discover markets and assets definitions, trading fees
* **Market Data**: Access real-time and historical market data, including prices, candles and order execution
* **Wallet Data**: Get information about the wallet's accounts, positions and orders
* **Order Entry**:  Create and cancel orders

## Signatures

For private endpoints that access user-specific data or perform actions on behalf of a user, authentication is required through wallet signatures. This ensures that only authorized users can create and update orders.

## Rate Limits

To ensure fair usage and optimal performance, the API implements rate limiting. For more information, see the [Rate Limits ](rest-api-reference/rate-limits.md)documentation.

For real-time data needs, consider using our [WebSocket API](/broken/pages/SL6x5f1Ngf6GtXYAOL10) for streaming updates.
