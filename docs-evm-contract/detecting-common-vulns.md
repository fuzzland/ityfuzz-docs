# Detecting Common Vulns

### Balance Extraction

_Confidence: 100%_

Detect whether the attackers can steal ETH / native tokens from the contract.

Example:

```solidity
function buggy() public {
    payable(msg.sender).transfer(1 ether);
}
```

### **Token Extraction**&#x20;

_Confidence: 100%_

Detect whether the attackers can steal ERC20 / ERC721 tokens from the contract, determined by the positive earnings of the attackers. The earning is calculated by liquidating the token on related Uniswap V2 pairs.&#x20;

Example:

```solidity
function buggy() public {
    // the price of tokenAddr2 is higher than tokenAddr1
    SomeToken(tokenAddr1).transferFrom(msg.sender, address(this), 1000);
    SomeToken(tokenAddr2).transfer(msg.sender, 1000);
}
```

### **Uniswap Pair Issues**&#x20;

_Confidence: Medium_

Identify misuse of Uniswap pair that could lead to price manipulation attacks.

Example:

```solidity
function buggy() public {
    burn(IUniswapPair(pairAddr), 1000000);
}
```

### **Arbitrary Selfdestruct**&#x20;

_Confidence: 100%_

Detect whether the contract can be selfdestructed by anyone.

Example:

```solidity
function buggy() public {
    selfdestruct(msg.sender);
}
```

### Integer Overflow / Underflow

_Confidence: Low_

Detect whether the contract add / multiply / subtract leading to overflow and underflow.&#x20;

Example:

```solidity
function buggy() public {
    return type(uint256).max * 2;
}

```
