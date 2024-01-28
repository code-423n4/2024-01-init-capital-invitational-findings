# setQuoteAsset and getBaseAssetAndQuote can be refractored to reduce code size

https://github.com/code-423n4/2024-01-init-capital-invitational/blob/a01c4de620be98f9e57a60cf6a82d4feaec54f58/contracts/hook/MarginTradingHook.sol#L431

```solidity
  function setQuoteAsset(address _tokenA, address _tokenB, address _quoteAsset) external onlyGovernor {
        _require(_tokenA != address(0) && _tokenB != address(0), Errors.ZERO_VALUE);
        _require(_quoteAsset == _tokenA || _quoteAsset == _tokenB, Errors.INVALID_INPUT);
        _require(_tokenA != _tokenB, Errors.NOT_SORTED_OR_DUPLICATED_INPUT);
        // sort tokenA and tokenB
        // @audi gas
        (address token0, address token1) = _tokenA < _tokenB ? (_tokenA, _tokenB) : (_tokenB, _tokenA);
        __quoteAssets[token0][token1] = _quoteAsset;
    }

    /// @inheritdoc IMarginTradingHook
    function getBaseAssetAndQuoteAsset(address _tokenA, address _tokenB)
        public
        view
        returns (address baseAsset, address quoteAsset)
    {
        // sort tokenA and tokenB
        (address token0, address token1) = _tokenA < _tokenB ? (_tokenA, _tokenB) : (_tokenB, _tokenA);
        quoteAsset = __quoteAssets[token0][token1];
        _require(quoteAsset != address(0), Errors.ZERO_VALUE);
        baseAsset = quoteAsset == token0 ? token1 : token0;
    }

```

as we can see the logic 

```solidity
        (address token0, address token1) = _tokenA < _tokenB ? (_tokenA, _tokenB) : (_tokenB, _tokenA);
```

is implemented several times,

 it is recommended to put the logic above to a seperate function and then call the function to sort token to gas

# remove the ETH wrapper logic to reduce gas consumption

https://github.com/code-423n4/2024-01-init-capital-invitational/blob/a01c4de620be98f9e57a60cf6a82d4feaec54f58/contracts/hook/MarginTradingHook.sol#L67

```solidity
   modifier depositNative() {
        if (msg.value != 0) IWNative(WNATIVE).deposit{value: msg.value}();
        _;
    }

    modifier refundNative() {
        _;
        // refund native token
        uint wNativeBal = IERC20(WNATIVE).balanceOf(address(this));
        // NOTE: no need receive function since we will use TransparentUpgradeableProxyReceiveETH
        if (wNativeBal != 0) IWNative(WNATIVE).withdraw(wNativeBal);
        uint nativeBal = address(this).balance;
        if (nativeBal != 0) {
            (bool success,) = payable(msg.sender).call{value: nativeBal}('');
            _require(success, Errors.CALL_FAILED);
        }
    }
```

this modifier applies to every functions,

but in the case when the borrow pool and collateral pool underlying token is not WETH,

such check stills runs and waste user's gas

it is recommended to remove such logic

the protocol can explicitly always let user wrap ETH to WETH beforehand and only handle WETH to avoid managing ETH -> WETH conversion to reduce the total instruction of smart contract and save gas.

# WLPMoeMasterChef reward token list can be outdated when master chef update extra reward contract and for loop runs unnecessarily

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

but the problem is that in this case, the reward token list will have [MOE, and WETH and JOE],

and the for loop always runs three times and the for loop that runs on WETH is considered wasting gas

https://github.com/code-423n4/2024-01-init-capital-invitational/blob/a01c4de620be98f9e57a60cf6a82d4feaec54f58/contracts/wrapper/WLpMoeMasterChef.sol#L71

it is recommended to dynamically remove and 
adjust ___rewardTokens length based on extra token reward to avoid unnecessary for loop run

