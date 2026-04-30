# Conditional Orders

ReyaDEX supports setting a “Stop Loss” or "Take Profit" conditional order against open positions. Traders can set a Stop Loss and Take Profit price for each position. When the market price reaches this trigger price, the position is closed by executing a Market Order of the same size but in the opposite direction of the current exposure. Stop Loss or Take Profit conditional orders can be canceled by traders at any time.

Traders can edit positions without adjusting the conditional orders against that position. The Conditional orders will apply to the whole position at the time of execution. If a position is closed, then the associated conditional orders will be canceled. If a trader has Long exposure in a given market and trades short (Sell-side) such that the net exposure is short after the trade, Reya assumes the original long exposure to have been closed and clears any conditional orders.

Against a Long (Buy-side) position in a given market, traders can set a stop loss (trigger) price below the current estimated execution price. Against a Short (Sell-side) position, traders can set a Stop Loss (trigger) price above the current estimated execution price. With Take Profit orders, long positions need a trigger price above the current market (estimated execution) price and short positions need a trigger below the current price.

To ensure traders remain in control of their funds, traders must first authorize Reya to execute Conditional orders on their behalf and then generate a signature for each conditional order placed. The order will only be executed if Reya has both authorization and a valid signature. This is for the safety of the trader. Traders only need to authorise Reya once per account, but will need to generate a signature for each new conditional order (so two if setting both a stop loss and take profit) or edit or cancel of existing order.

_ReyaDEX cannot guarantee that any specific price will be achieved in connection with a conditional order. Markets in digital tokens are underdeveloped and can be volatile and illiquid. Traders should consider the risks associated with digital token markets before deciding to invest._
