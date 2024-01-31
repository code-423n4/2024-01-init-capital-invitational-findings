## Init protocol update analys
### Update overview
In this update, Init protocol has implemented few new things:
- `MarginTradingHook` that allows users to create leveraged positions, manage them and create trigger orders to control loss/profit of the position.
- `WLpMoeMasterChef` contract to allow wrapping of merchant more LP tokens into erc721 tokens that can be used as collateral in the markets.

### Main contracts
- `MarginTradingHook`: contains all means to open leveraged position and be able to manage it until it is closed. This contract allows users to create trigger orders when they would like to reduce/close position. Once price is reached, then arbitrager will execute order to get a reward that is calculated by order's limit price. 
- `MoeSwapHelper`: is developed to help `MarginTradingHook` with tokens swapping on merchant moe.
- `WLpMoeMasterChef`: contract that allows user to wrap their merchant more LP tokens. This contract also tracks rewards for each pid and distributes them based on amount of LP that was deposited by user. Creates erc721 token for each new deposit. Users are free to unwrap their erc721 position or claim fees at any time. Positions created by `WLpMoeMasterChef` can be used as collateral in the markets.

### Admin abuse risks
- `WLpMoeMasterChef` doesn't provide any functions for admins except deployment variables.
- `MarginTradingHook` has `setQuoteAsset` function that can be called by governor role. This function allows to set quote asset for the trade pair and allows governor to break swapping related functionality.

### Systemic risks
- `Reliability of oracles`. Protocol depends on oracles to get prices for the assets. The `MarginTradingHook` needs oracle to calculate trigger price and in case of oracle failure filling order functionality will stop working. In case of incorrect prices reporting `MarginTradingHook` will make incorrect calculation which will lead to loss of funds for position owner or arbitrager.
- `Integration`. Init protocol integrates with merchant moe protocol, which creates additional risk for funds stuck there.

### Recommendations
- `Merchant Moe integration`: add ability to use `emergencyWithdraw` function from MASTER_CHEF contract. This will allow you to not claim rewards during withdraw and avoid rewards that are caused by them in case of emergency.
- `Mode change`: currently there is no ability to change mode for position inside `MarginTradingHook`. This could be good feature for users, so they can do that without a need to close position and get better borrow factor.

### Time spent:
12 hours