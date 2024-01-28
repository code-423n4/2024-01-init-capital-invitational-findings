| Number | Title                                                                                                            |
|--------|------------------------------------------------------------------------------------------------------------------|
| 1      | Set treasury missing check address(0)                                                                            |
| 2      | Lack of function way to collateralize and decollateralize using WLP in MarginTradingHook.sol                      |
| 3      | WLPMoeMasterChef reward token list can be outdated when master chef update extra reward contract                 |
| 4      | If the order.recipient is blocklisted, his order can never be fulfilled                                          |
| 5      | Order cannot be filed in certain case and lack of view function to know if the order can be fulfilled            |
| 6      | FillOrder may subject to reentrancy                                                                              |
| 7      | Flashloan does not charge fee                                                                                    |
| 8      | Lack of view function for user to query if the flashloan can be used                                             |



# set treasury missing check address(0)

the code should validate _treausry address is not address(0) when setting Treasury in [lending pool](https://github.com/code-423n4/2024-01-init-capital-invitational/blob/a01c4de620be98f9e57a60cf6a82d4feaec54f58/contracts/lending_pool/LendingPool.sol#L243)

```solidity
function setTreasury(address _treasury) external accrue onlyGovernor {
        treasury = _treasury;
        emit SetTreasury(_treasury);
    }
```

in fact by default the treasury address is not set, 

and accure interest would revert when [minting interest to treasury address(0)](https://github.com/code-423n4/2024-01-init-capital-invitational/blob/a01c4de620be98f9e57a60cf6a82d4feaec54f58/contracts/lending_pool/LendingPool.sol#L160)

https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/f6f59bf10f40e06111c66e2fadf652a47919d2f6/contracts/token/ERC20/ERC20Upgradeable.sol#L251

```solidity
function _mint(address account, uint256 value) internal {
	if (account == address(0)) {
		revert ERC20InvalidReceiver(address(0));
	}
	_update(address(0), account, value);
}
```

# Lack of function way to collateralize and decollateralize using WLP in MarginTradingHook.sol

there lack of functionality to to collateralize and decollateralize using WLP in [MarginTradingHook.sol](https://github.com/code-423n4/2024-01-init-capital-invitational/blob/a01c4de620be98f9e57a60cf6a82d4feaec54f58/contracts/hook/MoneyMarketHook.sol#L24)

# WLPMoeMasterChef reward token list can be outdated when master chef update extra reward contract

when we updateRewards, the code would [append the extra reward token to reward token list](https://github.com/code-423n4/2024-01-init-capital-invitational/blob/a01c4de620be98f9e57a60cf6a82d4feaec54f58/contracts/wrapper/WLpMoeMasterChef.sol#L65)

```solidity
    // add extraReward to __rewardTokens set if there is extraRewarder
        (address extraRewarder, address extraRewardToken) = _getExtraRewarderAndToken(_pid);
        if (extraRewarder != address(0)) __rewardTokens[_pid].add(extraRewardToken);
```

and the reward token is never removed

but the problem is that the master chef admin can [remove extra reward or update extra reward](https://github.com/traderjoe-xyz/moe-core/blob/5eb20a10cbe4ee01f8db20da950309cb297e3c09/src/MasterChef.sol#L417)

suppose the reward token list has [MOE, and WETH], WETH is the extra reward

then the extra reward changes to [JOE],

then WETH token data is considered outdated.

# if the order.recipient is blocklisted, his order can never be fulfilled

when creating a order in margin trading hook, the code [record order.recipient](https://github.com/code-423n4/2024-01-init-capital-invitational/blob/a01c4de620be98f9e57a60cf6a82d4feaec54f58/contracts/hook/MarginTradingHook.sol#L496)

certain token support blocklisting behavior

https://github.com/d-xo/weird-erc20?tab=readme-ov-file#tokens-with-blocklists

but when filling the order, if the order.recipient is blocklisted, the order can never be fulfilled and fill order always revert

# Order cannot be filed in certain case and lack of view function to know if the order can be fulfilled

in margin trading hook, after user open margin position, user can create stop loss or take profits order

but if a user wants to fill the order, it is diffcult to know if the order can be fulfilled

querying whether the order status is active is not enough,

the order fill can revert for many reason,

it could be possible that the order.collAmount is oudated, the user already adjust margin position and remove most of the collateral

it could be possible that order.recipient is blocklisted and token transfer to order.recipient will fail and revert

it could be possible that the user is liquidated and there is no collateral left so when the order filler [remove collateral](https://github.com/code-423n4/2024-01-init-capital-invitational/blob/a01c4de620be98f9e57a60cf6a82d4feaec54f58/contracts/hook/MarginTradingHook.sol#L396) in exchange for filling order

it is recommendated to add a view function to preview if an order can be fulfilled or not

it is also recommended that the order that cannot be filled by user should be canceled

# fillOrder may subject to reentrancy

the function [fillOrder](https://github.com/code-423n4/2024-01-init-capital-invitational/blob/a01c4de620be98f9e57a60cf6a82d4feaec54f58/contracts/hook/MarginTradingHook.sol#L372) set order status as filled after external token transfer

https://docs.openzeppelin.com/contracts/3.x/erc777

this opens room for reentrancy if the token is ERC777 token when external user can control the execution flow via the receive hook

the external token transfer is this [line of code](https://github.com/code-423n4/2024-01-init-capital-invitational/blob/a01c4de620be98f9e57a60cf6a82d4feaec54f58/contracts/hook/MarginTradingHook.sol#L391)

```solidity
// transfer order owner's desired tokens to the specified recipient
       IERC20(order.tokenOut).safeTransferFrom(msg.sender, order.recipient, amtOut);
```

where order.recipient can re-enter the fillOrder to fill order more than one times

it is recommende to add nonReentrant modifer to the function fillOrder

# flashloan does not charge fee

when [flashloan](https://github.com/code-423n4/2024-01-init-capital-invitational/blob/a01c4de620be98f9e57a60cf6a82d4feaec54f58/contracts/core/InitCore.sol#L365), there is no fee charged, so user can flash loan for free

it is recommended to charge flash loan fee

# lack of view function for user to query if the flashloan can be used

it is recommended to implement and adopt 

https://eips.ethereum.org/EIPS/eip-3156

where there are view function for external user to call to query if the flashloan can be used

```solidity
   function maxFlashLoan(
        address token
    ) external view returns (uint256);

    /**
     * @dev The fee to be charged for a given loan.
     * @param token The loan currency.
     * @param amount The amount of tokens lent.
     * @return The amount of `token` to be charged for the loan, on top of the returned principal.
     */
    function flashFee(
        address token,
        uint256 amount
    ) external view returns (uint256);
```

