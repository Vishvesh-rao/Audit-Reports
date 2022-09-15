
# Velodrome C4 Contest Findings

## Math.sol (CBRT function)
- **Details**: Math.cbrt(uint256) (libraries/Math) is never used and should be removed
- **Github Link**: [L22](https://github.com/code-423n4/2022-05-velodrome/blob/7fda97c570b758bbfa7dd6724a336c43d4041740/contracts/contracts/libraries/Math.sol#L22)
- **Mitigation**: Comment out that function
## Inconsistent documentation with code
- **File**: `contracts/contracts/VotingEscrow.sol`
- **Summary**: Comments documenting code does not match the code functionality.
- **Details**: Comments in functions `_balance`, `balanceOf` are claiming, that “Throws if `_owner` is the zero address”. However there is no assertion for `_owner` being non-zero address. This may create assumptions of throwing for zero address in other functions, and create unpredicted behavior.
- **Github Links**: [L191](https://github.com/code-423n4/2022-05-velodrome/blob/7fda97c570b758bbfa7dd6724a336c43d4041740/contracts/contracts/VotingEscrow.sol#L191), [L198](https://github.com/code-423n4/2022-05-velodrome/blob/7fda97c570b758bbfa7dd6724a336c43d4041740/contracts/contracts/VotingEscrow.sol#L198)
- **Mitigation**: Add checks that require non-zero address.
## Native Velo token zero addresses
- **File**: `contracts/contracts/Velo.sol`
- **Summary**: No input check in functions.
- **Details**: No functions check for address non-zero validation. It is possible to mint, send and approve to zero address.
- **Github Links**: [L36](https://github.com/code-423n4/2022-05-velodrome/blob/7fda97c570b758bbfa7dd6724a336c43d4041740/contracts/contracts/Velo.sol#L36), [L42](https://github.com/code-423n4/2022-05-velodrome/blob/7fda97c570b758bbfa7dd6724a336c43d4041740/contracts/contracts/Velo.sol#L42), [L49](https://github.com/code-423n4/2022-05-velodrome/blob/7fda97c570b758bbfa7dd6724a336c43d4041740/contracts/contracts/Velo.sol#L49)
- **Mitigation**: Provide checks for non-zero addresses.
## Velo token race condition
- **File**: `contracts/contracts/Velo.sol`
- **Summary**: Possible attack on native Velo token using approve and transferFrom methods.
- **Details**: 3. Velo token does not inherit from ERC20 standard token, and only possible function for allowance is `approve`. Using this function is known to be insecure, it is possible to transfer more tokens than it should be possible.
Possible attack scenario:
    1. Alice approves Bob to transfer her N amount of tokens, calling `approve(bobAddress,N)`.
    2. After some time Alice wants to change the allowance to different amount M. She calls `approve(bobAddress, M)`.
    3. Bob detects Alices call before it was mined (he finds in mempool or just know about it). He calls `transferFrom(aliceAddress, bobAddress, N)` with higher gas price than Alice’s second transaction.
    4. If Bob’s transaction will be executed before Alice’s, he will transfer N tokens and will be approved for M amount.
    5. Bob calls `transferFrom(aliceAddress, bobAddress, M)` right after Alice calls second approval.
    
    Bob was able to transfer N + M tokens, while Alice allowed only for N + (M-N).
- **Github Links**: [L36](https://github.com/code-423n4/2022-05-velodrome/blob/7fda97c570b758bbfa7dd6724a336c43d4041740/contracts/contracts/Velo.sol#L36), [L60](https://github.com/code-423n4/2022-05-velodrome/blob/7fda97c570b758bbfa7dd6724a336c43d4041740/contracts/contracts/Velo.sol#L60)
- **Mitigation**: ERC20 token should inherit (from ERC20 standard) functions `increaseAllowance` and `decreaseAllowance` which were designed to mitigate this unwanted behavior.
## Unchecked reciever address
- **File**: L49, L60 in [contracts/contracts/Velo.sol]
- **Summary**:Missing receiver’s address validation checks in ```_transfer()``` and ```transferFrom()``` functions.
- **Details**:If the receiver address provided in ```_transfer()``` and ```transferFrom() ```function is the address of the contract itself then it may result in a forever loss of tokens and sent tokens will get stuck in the smart contract.
- **Mitigation:** creating a modifier validating that the `_to` address is neither 0x0 nor the smart contract's own address.
## Use of strict equality with ```block.timestamp```
- **Files:** 
- [L243](https://github.com/code-423n4/2022-05-velodrome/blob/7fda97c570b758bbfa7dd6724a336c43d4041740/contracts/contracts/Pair.sol#L243) - `Pair.sol`
- [L78](https://github.com/code-423n4/2022-05-velodrome/blob/7fda97c570b758bbfa7dd6724a336c43d4041740/contracts/contracts/RewardsDistributor.sol#L78) - `RewardsDistributor.sol`
- [L656](https://github.com/code-423n4/2022-05-velodrome/blob/7fda97c570b758bbfa7dd6724a336c43d4041740/contracts/contracts/VotingEscrow.sol#L656) - `VotingEscrow.sol`
- **Summary:** use of strict equality with `block.timestamp` may not work as intended.
- **Details:** using strict equality with `block.timestamp` would not be safe, since a block with that exact timestamp may never get mined. So, it may be possible for certain code blocks using strict equality with `block.timestamp` will never get executed and this will cause unintended behavior of the smart contract.
- **Mitigation:** should avoid the use of strict equality with `block.timestamp`
## No emitted event in state-changing functions
- **Files:**  
    1. `contracts/contracts/Bribe.sol`
    2. `contracts/factories/BribeFactory.sol`
    3. `contracts/factories/GaugeFactory.sol`
    4. `contracts/factories/PairFactory.sol`
- **Details:**  State-changing functions: `setGauge`, `addRewardToken`, `swapOutRewardToken` `deliverReward` in `Bribe.sol` ; `createBribe` in `BribeFactory.sol` ; `setTeam` , `createGauge` and `createGaugeSingle`  in `GaugeFactory` ; and `setPauser` , `acceptPauser` , `setPause` , `setFeeManager` , `acceptFeeManager` `setFee` in `PairFactory.sol` do not emit requisite events
- **Github Link:**
- [L30](https://github.com/code-423n4/2022-05-velodrome/blob/main/contracts/contracts/Bribe.sol) - `Bribe.sol`
- [L66](https://github.com/code-423n4/2022-05-velodrome/blob/main/contracts/contracts/Bribe.sol) - `Bribe.sol`
- [L76](https://github.com/code-423n4/2022-05-velodrome/blob/main/contracts/contracts/Bribe.sol) - `Bribe.sol`
- [L83](https://github.com/code-423n4/2022-05-velodrome/blob/main/contracts/contracts/Bribe.sol) - `Bribe.sol`
- [L9](https://github.com/code-423n4/2022-05-velodrome/blob/main/contracts/contracts/factories/BribeFactory.sol) - `BribeFactory.sol`
- [L17](https://github.com/code-423n4/2022-05-velodrome/blob/main/contracts/contracts/factories/GaugeFactory.sol) - `GaugeFactory.sol`
- [L22](https://github.com/code-423n4/2022-05-velodrome/blob/main/contracts/contracts/factories/GaugeFactory.sol) - `GaugeFactory.sol`
- [L28](https://github.com/code-423n4/2022-05-velodrome/blob/main/contracts/contracts/factories/GaugeFactory.sol) - `GaugeFactory.sol`
- [L40](https://github.com/code-423n4/2022-05-velodrome/blob/main/contracts/contracts/factories/PairFactory.sol) - `PairFactory.sol`
- [L45](https://github.com/code-423n4/2022-05-velodrome/blob/main/contracts/contracts/factories/PairFactory.sol) - `PairFactory.sol`
- [L50](https://github.com/code-423n4/2022-05-velodrome/blob/main/contracts/contracts/factories/PairFactory.sol) - `PairFactory.sol`
- [L55](https://github.com/code-423n4/2022-05-velodrome/blob/main/contracts/contracts/factories/PairFactory.sol) - `PairFactory.sol`
- [L60](https://github.com/code-423n4/2022-05-velodrome/blob/main/contracts/contracts/factories/PairFactory.sol) - `PairFactory.sol`
- [L65](https://github.com/code-423n4/2022-05-velodrome/blob/main/contracts/contracts/factories/PairFactory.sol) - `PairFactory.sol`
- **Mitigation:** Consider creating appropriate events and emitting same in state-changing functions
## No validation logic to check zero address
- **File**: `contracts/contracts/Bribe.sol`
- **Details**:  `addRewardToken` function lacks validation logic to ensure that zero-address is not - **Github Link:** [L66](https://github.com/code-423n4/2022-05-velodrome/blob/main/contracts/contracts/Bribe.sol)
- **Mitigation:** Consider writing logic to guard against zero address `require(token != address(0);`