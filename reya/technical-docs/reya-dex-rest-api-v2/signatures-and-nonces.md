# Signatures and Nonces

Signatures are required to safely execute orders in the name of your wallet. The API endpoints for order creation require a user signature that must be created on the client side using your private key. These signatures are validated on-chain for all order details you specified to ensure no unintended action is executed on your account. Internal nonces for your wallet address, tracked in the smart contracts, are used to prevent replay attacks.

## Nonces

The internal nonces are different from Ethereum wallet nonce. These are instead tracked in Reya DEX's smart contracts under unordered `uint256` values. Nonces are generated based on a combination of account ID, market ID, and timestamp, which ensures uniqueness for each action.

### Nonce Generation

The Python SDK provides a method to generate nonces automatically:

```python
def create_orders_gateway_nonce(self, account_id: int, market_id: int, timestamp_ms: int) -> int:
    """Create a nonce for Orders Gateway orders."""
    # Validate the input ranges
    if market_id < 0 or market_id >= 2**32:
        raise ValueError("marketId is out of range")
    if account_id < 0 or account_id >= 2**128:
        raise ValueError("accountId is out of range")
    if timestamp_ms < 0 or timestamp_ms >= 2**64:
        raise ValueError("timestamp is out of range")

    hash_uint256 = (account_id << 98) | (timestamp_ms << 32) | market_id

    return hash_uint256
```

This function takes three parameters:

* `account_id`: Your account ID on Reya DEX
* `market_id`: The ID of the market for the order. This is required and is an alternative for the symbol used in the API. To get the market ID for a symbol, use the endpoint: [https://api.reya.xyz/v2/marketDefinitions](https://api.reya.xyz/v2/marketDefinitions)&#x20;
* `timestamp_ms`: Current timestamp in milliseconds, which ensures the nonce is unique even if other parameters are reused

### On-Chain Nonce Validation

When an order with a signature is submitted, the on-chain smart contracts verify that the nonce has not been used before. Once a signature with a specific nonce successfully executes a transaction, that nonce is marked as "used" in the contract. Any subsequent attempt to use the same nonce will fail, preventing replay attacks.

With the pattern of using millisecond timestamps as part of the nonce generation, there should be sufficient for no overlaps in normal usage. Each nonce is unique to the specific combination of account, market, and timestamp. This format is recommended but can be modified if the use-case requires it.

## Signatures

For order creation, Reya DEX requires a signature from the account owner to confirm the intended order details match what is received by the contracts at execution. This is to prevent any unintended action from being executed on your account.

Reya DEX uses the [EIP-712](https://eips.ethereum.org/EIPS/eip-712) standard for structured data signing. This standard provides a secure way to sign typed structured data, making it more readable and secure compared to raw message signing.

The Python SDK offers a good [example](https://github.com/Reya-Labs/reya-python-sdk/blob/main/sdk/reya_rest_api/auth/signatures.py) of what values are required to be signed and how signatures are generated.

### Domain Structure

The following is the domain required as part of the EIP-712 structure:

```python
domain = {
    "name": "Reya", 
    "version": "1", 
    "verifyingContract": self.config.default_orders_gateway_address
}
```

### Type Definitions for Order Creation

Order creation uses the following type structure:

```python
types = {
    "ConditionalOrder": [
        {"name": "verifyingChainId", "type": "uint256"},
        {"name": "deadline", "type": "uint256"},
        {"name": "order", "type": "ConditionalOrderDetails"},
    ],
    "ConditionalOrderDetails": [
        {"name": "accountId", "type": "uint128"},
        {"name": "marketId", "type": "uint128"},
        {"name": "exchangeId", "type": "uint128"},
        {"name": "counterpartyAccountIds", "type": "uint128[]"},
        {"name": "orderType", "type": "uint8"},
        {"name": "inputs", "type": "bytes"},
        {"name": "signer", "type": "address"},
        {"name": "nonce", "type": "uint256"},
    ],
}
```

### Order Type Inputs Encoding

The `inputs` field in the order structure requires different encoding based on the order type:

1.  **Limit Order**:

    ```
    int256 base, uint256 limitPrice
    ```
2.  **Trigger Order**:

    ```
    bool is_buy, uint256 trigger_price, uint256 limit_price
    ```

The inputs for each action need to be encoded into bytes and are different for each order type. The SDK handles this encoding along with the message signature.

## Cancelling signatures

Similar to creation, cancelling \[[`https://api.reya.xyz/v2/cancelOrder`](https://api.reya.xyz/v2/cancelOrder)] an open order requires a signature. This is not EIP712, but a simple signed message:

```python
 cancel_message = {
            "orderId": your_order_id,
            "status": "cancelled",
            "actionType": "changeStatus",
 }
```

The Python SDK provides a nice [example](https://github.com/Reya-Labs/reya-python-sdk/blob/276357727261e540dd34fcb01a71a8e190c0ecf6/sdk/reya_rest_api/auth/signatures.py#L171) of how to create this signature.
