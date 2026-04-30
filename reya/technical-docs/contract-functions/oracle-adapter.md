---
description: >-
  Receives and track the latest price updates given by a trusted off-chain
  source.
---

# Oracle Adapter

#### getLatestPricePayload <a href="#getlatestpricepayload" id="getlatestpricepayload"></a>

```
function getLatestPricePayload(string memory assetPairId) external view returns (StorkPricePayload memory)
```

<details>

<summary>StorkSignedPayload</summary>

```
struct StorkSignedPayload {
    address oraclePubKey;
    StorkPricePayload pricePayload;
    bytes32 r;
    bytes32 s;
    uint8 v;
}

struct StorkPricePayload {
    string assetPairId;
    uint256 timestamp;
    uint256 price;
}
```

</details>

Returns the latest price payload for the given Stork asset pair ID (e.g 'AAVEUSDMARK' for futures prices of Aave). This payload has been signed by a trusted source and gives the latest price used by the Perpetuals Instrument as the reference price for match orders.

#### fulfillOracleQuery <a href="#fulfilloraclequery" id="fulfilloraclequery"></a>

```
function fulfillOracleQuery(bytes calldata signedOffchainData) external payable override
```

Updates the latest reference price of the asset pair. The `signedOffchainData` is a bytes encoding of the `StorkSignedPayload`. This is a payload of the latest price details with a signature given by a trusted source. The signature is verified against the received data and must match for the new price to be accepted.

Anyone can update the prices at any point, the only constraint is that they have to be signed by a trusted source. One can subscribe to price updates from this source and send `fulfillOracleQuery` transactions with the latest prices. It is recommended to append updates for every market before executing a match order or a liquidation (e.g. by using a Multicall [contract](https://app.gitbook.com/o/PWuj4sk8IMHDHbNTNmMS/s/b3btfuLSfJJJXBJGFc5f/~/changes/1/contract-addresses)), such that the Core operations use the latest prices.

Submit signed payload of price update.
