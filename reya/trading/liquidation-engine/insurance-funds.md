# Insurance Funds

Reya Network includes a system of insurance funds to backstop potential collateral pool insolvencies. These insurance funds accumulate the greater part of the liquidation fees, and fill the shortfall when losses exceeded the deposited margin. But two important aspects:

1. There are set limits on how much the insurance funds can cover (depending on size of account). ADL at a loss would kick in beyond that limit.
2. The insurance funds plug in the shortfall, but only **realized shortfalls resulting from the liquidation** (i.e., measured as the realized loss of the liquidated account resulting from the liquidation)
3. The insurance funds plug the shortfall to the pool as whole and not to any particular account.

A development of the full ecosystem of exchanges on top of Reya Network will most likely introduce a second layer of backstop after the insurance funds.
