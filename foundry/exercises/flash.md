# Flash Exercises

In this exercise, you'll learn how to liquidate an under-collateralized debt from Aave V3.

The starter code for this exercise is provided in `foundry/src/exercises/Flash.sol`

Solution is provided in `foundry/src/solutions/Flash.sol`

## Task 1 - Initiate flash loan

```solidity
function flash(address token, uint256 amount) public {
    pool.flashLoanSimple({
        receiverAddress: address(this),
            asset: token,
            amount: amount,
            params: abi.encode(msg.sender),
            referralCode: 0

    })
}
```

## Task 2 - Repay flash loan

```solidity
function executeOperation(
    address asset,
    uint256 amount,
    uint256 fee,
    address initiator,
    bytes calldata params
) public returns (bool) {
    require(msg.sender == address(pool), "not authorized");
        require(initiator == address(this), "invalid initiator");

        address caller = abi.decode(params, (address));
        IERC20(asset).transferFrom(caller, address(this), fee);

        IERC20(asset).approve(msg.sender, amount + fee);

        return true;
}
```

Implement the callback function called by Aave.

> Hints
>
> - ABI decode `params` and transfer the flash loan fee from the caller that is encoded into `params`

## Test

```shell
forge test --fork-url $FORK_URL --fork-block-number $FORK_BLOCK_NUM --match-path test/Flash.test.sol -vvv
```
