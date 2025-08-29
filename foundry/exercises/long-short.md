# Long and Short Exercises

In this exercise, you'll learn how to create a long and short position with Aave V3.

The starter code for this exercise is provided in `foundry/src/exercises/LongShort.sol`

Solution is provided in `foundry/src/solutions/LongShort.sol`

## Task 1 - Open a long or a short position

```solidity
function open(OpenParams memory params)
    public
    returns (uint256 collateralAmountOut)
{
    require(params.minHealthFactor > 1e18, "min health factor <= 1");

        // Transfer collateral
        IERC20(params.collateralToken).transferFrom(
            msg.sender, address(this), params.collateralAmount
        );

        // Supply collateral
        IERC20(params.collateralToken).approve(
            address(pool), params.collateralAmount
        );
        supply(params.collateralToken, params.collateralAmount, msg.sender);

        // Borrow token
        borrow(params.borrowToken, params.borrowAmount, msg.sender);

        // Check health factor
        require(
            getHealthFactor(msg.sender) >= params.minHealthFactor,
            "health factor < min"
        );

        // Swap borrowed token to collateral token
        IERC20(params.borrowToken).approve(address(router), params.borrowAmount);
        return swap({
            tokenIn: params.borrowToken,
            tokenOut: params.collateralToken,
            amountIn: params.borrowAmount,
            amountOutMin: params.minSwapAmountOut,
            receiver: msg.sender,
            data: params.swapData
        });
}
```

Implement the `open` function which will deposit the collateral token into Aave, borrow another token and then swap it for more collateral token.

> Hints:
>
> - Call functions inside `Aave` and `Swap` contracts

## Task 2 - Close a long or a short position

```solidity
function close(CloseParams memory params)
    public
    returns (
        uint256 collateralWithdrawn,
        uint256 debtRepaidFromMsgSender,
        uint256 borrowedLeftover
    )
{
   // Transfer collateral
        IERC20(params.collateralToken).transferFrom(
            msg.sender, address(this), params.collateralAmount
        );

        // Swap collateral to borrowed token
        IERC20(params.collateralToken).approve(
            address(router), params.collateralAmount
        );
        uint256 amountOut = swap({
            tokenIn: params.collateralToken,
            tokenOut: params.borrowToken,
            amountIn: params.collateralAmount,
            amountOutMin: params.minSwapAmountOut,
            receiver: address(this),
            data: params.swapData
        });

        // Repay borrowed token
        uint256 debtToRepay = Math.min(
            getVariableDebt(params.borrowToken, msg.sender),
            params.maxDebtToRepay
        );
        IERC20(params.borrowToken).approve(address(pool), debtToRepay);
        uint256 repayAmount = 0;
        if (debtToRepay > amountOut) {
            // msg.sender repays for the difference
            repayAmount = debtToRepay - amountOut;
            IERC20(params.borrowToken).transferFrom(
                msg.sender, address(this), repayAmount
            );
        }
        repay(params.borrowToken, debtToRepay, msg.sender);

        // Withdraw collateral to msg.sender
        IERC20 aToken = IERC20(getATokenAddress(params.collateralToken));
        aToken.transferFrom(
            msg.sender,
            address(this),
            Math.min(
                aToken.balanceOf(msg.sender), params.maxCollateralToWithdraw
            )
        );

        uint256 withdrawn = withdraw(
            params.collateralToken, params.maxCollateralToWithdraw, msg.sender
        );

        // Transfer profit = swapped - repaid
        uint256 bal = IERC20(params.borrowToken).balanceOf(address(this));
        if (bal > 0) {
            IERC20(params.borrowToken).transfer(msg.sender, bal);
        }

        return (withdrawn, repayAmount, bal);
}
```

Implement the `close` function which will transfer the collateral token from `msg.sender`, swap it with the borrowed token, repay debt to Aave and withdraw the collateral and profit from this strategy.

> Hints:
>
> - Call functions inside `Aave` and `Swap` contracts

## Test

```shell
forge test --fork-url $FORK_URL --fork-block-number $FORK_BLOCK_NUM --match-path test/LongShort.test.sol -vvv
```
