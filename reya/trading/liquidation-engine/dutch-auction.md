# Dutch auction

The Dutch auction process kicks in either when the open bidding process has failed to bring the account back to health, or when the liquidation margin requirement was breached directly and there was no time for the preparatory bidding. Recognizing the urgency of such cases, the Protocol consecutively quotes decreasing $$d$$, starting at some predefined maximum, and ending at a predefined minimum, and immediately allows execution of liquidation proposals with margin reward parameter smaller than $$d$$.
