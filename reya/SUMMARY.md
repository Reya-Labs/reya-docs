# Table of contents

## Get Started

* [What Is Reya](README.md)
* [Why Reya Exists](get-started/why-reya-exists.md)
* [Who Is Behind Reya](get-started/who-is-behind-reya.md)
* [How Reya Is Different](get-started/how-reya-is-different.md)
* [Architecture (at a glance)](get-started/architecture-at-a-glance.md)
* [Reya Roadmap](get-started/reya-roadmap.md)

## Trading

* [Derivatives Clearing](trading/derivatives-clearing.md)
* [Perpetual Futures](trading/perpetual-futures.md)
* [Cross-margining](trading/cross-margining.md)
* [Margin System](trading/margin-system.md)
* [Cross-collateralization](trading/cross-collateralization/README.md)
  * [Haircuts](trading/cross-collateralization/haircuts.md)
  * [Auto-exchange conditions](trading/cross-collateralization/auto-exchange-conditions.md)
  * [Auto-exchange mechanism](trading/cross-collateralization/auto-exchange-mechanism.md)
* [Liquidation Engine](trading/liquidation-engine/README.md)
  * [Ranked liquidations](trading/liquidation-engine/ranked-liquidations.md)
  * [Dutch auction](trading/liquidation-engine/dutch-auction.md)
  * [Backstop LPs](trading/liquidation-engine/backstop-lps.md)
  * [Insurance Funds](trading/liquidation-engine/insurance-funds.md)
* [Automatic leveraging of PnL across instruments](trading/automatic-leveraging-of-pnl-across-instruments.md)
* [Spot markets](trading/spot-markets.md)
* [Reya Governance](trading/reya-governance.md)
* [The Liquidity Layer](trading/the-liquidity-layer/README.md)
  * [Reya’s pool for perpetual futures](trading/the-liquidity-layer/reyas-pool-for-perpetual-futures.md)
  * [Offering liquidity in the passive pool](trading/the-liquidity-layer/offering-liquidity-in-the-passive-pool.md)
  * [Deepdive into passive perps](trading/the-liquidity-layer/deepdive-into-passive-perps.md)
  * [Network Owned Liquidity](trading/the-liquidity-layer/network-owned-liquidity.md)
* [Reya DEX](trading/reya-dex/README.md)
  * [Auto-exchange and Liquidations](trading/reya-dex/auto-exchange-and-liquidations.md)
  * [Conditional Orders](trading/reya-dex/conditional-orders.md)
  * [Collateral liquidations](trading/reya-dex/collateral-liquidations.md)
  * [Settlement and cross-collateralization](trading/reya-dex/settlement-and-cross-collateralization.md)
  * [Margin](trading/reya-dex/margin.md)
* [Reya Network](trading/reya-network/README.md)
  * [The structure of Reya Network](trading/reya-network/the-structure-of-reya-network.md)
  * [Who's Behind Reya Network](trading/reya-network/whos-behind-reya-network.md)

## Chain

* [Introduction](chain/introduction.md)

## REYA Token

* [Context](reya-token/context.md)
* [Token Design](reya-token/token-design/README.md)
  * [Network Security and Economic Incentives](reya-token/token-design/network-security-and-economic-incentives.md)
  * [Monetary Policy](reya-token/token-design/monetary-policy.md)
  * [Governance](reya-token/token-design/governance.md)
  * [sREYA Additional Utility](reya-token/token-design/sreya-additional-utility.md)
* [Allocation & Release Schedule](reya-token/allocation-and-release-schedule.md)
* [Reya Chain Points: FAQs](reya-token/reya-chain-points-faqs.md)

## Native Stablecoin

* [Introduction](native-stablecoin/introduction.md)
* [srUSD](native-stablecoin/srusd.md)
* [FAQs](native-stablecoin/faqs.md)

## Security <a href="#technical-docs" id="technical-docs"></a>

* [Reya DEX REST API v2](technical-docs/reya-dex-rest-api-v2/README.md)
  * ```yaml
    type: builtin:openapi
    props:
      models: true
    dependencies:
      spec:
        ref:
          kind: openapi
          spec: specsendpoint
    ```
  * [Rate Limits](technical-docs/reya-dex-rest-api-v2/rate-limits.md)
  * [Signatures and Nonces](technical-docs/reya-dex-rest-api-v2/signatures-and-nonces.md)
* [Reya DEX Websocket API V2](technical-docs/reya-dex-websocket-api-v2.md)
* [Contract Functions](technical-docs/contract-functions/README.md)
  * [Core](technical-docs/contract-functions/core.md)
  * [Passive Perp Instrument](technical-docs/contract-functions/passive-perp-instrument.md)
  * [Passive Pool](technical-docs/contract-functions/passive-pool.md)
  * [Oracle Adapter](technical-docs/contract-functions/oracle-adapter.md)
* [Contract Addresses](technical-docs/contract-addresses.md)
* [Smart Contract Withdrawals](technical-docs/smart-contract-withdrawals.md)
* [Audits](technical-docs/audits.md)

## Resources

* [Join the Reya Community](https://chat.reya.xyz/)
* [x.com](https://x.com/reya_xyz)
* [Blog](https://blog.reya.network/)
* [Block Explorer](https://explorer.reya.network/)
