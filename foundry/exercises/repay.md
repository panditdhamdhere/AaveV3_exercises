# Repay Exercise

In this exercise, you'll learn how to repay tokens to Aave V3.

The starter code for this exercise is provided in `foundry/src/exercises/Repay.sol`

Solution is provided in `foundry/src/solutions/Repay.sol`

## Task 1 - Repay all the debt owed to Aave V3

```solidity
function repay(address token) public returns (uint256) {
    uint256 balance = IERC20(token).balanceOf(address(this));
    uint256 debt = getVariableDebt(token);

    if (debt > balance) {
        IERC20(token).transferFrom(msg.sender, addrees(this), debt - balance);
    }

    IERC20(token).approve(address(pool), debt);

    uint256 repaid = pool.repay({
        asset: token,
        amount: type(uint256).max,
        interestRateMode: 2,
        onBehalfOf: address(this)
    });

    return repaid;
}
```

Implement the `repay` function to repay `token` to Aave.

- `token` is the token to repay
- Pay all of debt that this contract owes to Aave

> Hint - Call `pool.repay`

## Test

```shell
forge test --fork-url $FORK_URL --fork-block-number $FORK_BLOCK_NUM --match-path test/Repay.test.sol -vvv
```
