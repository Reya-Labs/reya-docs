---
description: Liquidity pool that acts as a counter-party for all trades on the Reya DEX.
---

# Passive Pool

#### getPoolQuoteToken <a href="#getpoolquotetoken" id="getpoolquotetoken"></a>

```
function getPoolQuoteToken(uint128 id) external view returns (address quoteToken);
```

Given the pool id, it returns the liquidity token address. Currently there is only 1 pool, id = 1, and its quote token is rUSD.

#### getPoolAccountId <a href="#getpoolaccountid" id="getpoolaccountid"></a>

```
function getPoolAccountId(uint128 id) external view returns (uint128 accountId);
```

Given the pool id, it returns the pool's margin account id registered in Core.

#### addLiquidity <a href="#addliquidity" id="addliquidity"></a>

```
function addLiquidity(uint128 poolId, address owner, uint256 amount, uint256 minShares) external returns (uint256);
```

Transfers the specified `amount` of liquidity from the caller to the pool and, based on the current price, mints a corresponding amount of shares to the given 'owner' address. For example, if 3 rUSD is deposited (3\* 10^6 with precision) and the share per share is 1.5 (1.2 \* 10^18 with precision), then the amount of shares received will be 3/1.5 = 2 shares (2 \* 10^30 with precision).

#### removeLiquidity <a href="#removeliquidity" id="removeliquidity"></a>

```
function removeLiquidity(uint128 poolId, uint256 sharesAmount, uint256 minOut) external returns (uint256 tokenAmount);
```

Burns the specified `sharesAmount` of shares and, based on the current price, withdraws the corresponding amount of liquidity to the caller's address. 'minOut' is the minimum amount of quote tokens expected to be received by the caller.

#### removeLiquidityBySig <a href="#removeliquiditybysig" id="removeliquiditybysig"></a>

```
function removeLiquidityBySig(address owner, uint128 poolId, uint256 sharesAmount, uint256 minOut, EIP712Signature memory sig, bytes memory extraSignatureData) external returns (uint256 tokenAmount);
```

Additionally to `removeLiquidity`, this function allows liquidity removal from a user's account by validating their signature instead of validating msg.sender. It sends the funds to the given 'owner'.

#### getAccountBalance <a href="#getaccountbalance" id="getaccountbalance"></a>

```
function getAccountBalance(uint128 poolId, address account) external view returns (uint256);
```

Given the user’s address, it returns the user's share balance in the specified pool. Shares have precision 30.

#### getPoolMarginBalance <a href="#getpoolmarginbalance" id="getpoolmarginbalance"></a>

```
function getPoolMarginBalance(uint128 poolId) external view returns (uint256);
```

Returns the margin balance of the pool margin account in Core in the pool’s quote token.

#### getSharePrice <a href="#getshareprice" id="getshareprice"></a>

```
function getSharePrice(uint128 poolId) external view returns (UD60x18)
```

This is the margin balance of the pool divided by the share supply. Note, the precision of the margin balance is adjusted from the quote token decimals to 30 decimals precision. The price has 18 decimals precision.

#### getShareSupply <a href="#getsharesupply" id="getsharesupply"></a>

```
function getShareSupply(uint128 poolId) external view returns (uint256)
```

Returns the total amount of shares tracked by the pool. When adding liquidity this amount increases by the number of shares minted based on the share price and the liquidity provided. When removing liquidity, supply decreases by the amount of shares burnt. The share supply has 30 decimals precision.
