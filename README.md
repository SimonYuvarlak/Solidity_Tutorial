# About This Solidity_Tutorial
This is a Solidity tutorial for beginners which uses an Proposal Smart Contract to teach Solidity Smart Contract Development basics.
To understand the course, you need basic Solidity knowledge including:
- `Function Modifiers`
- `Data types`
- `Function Types`
- `Storage`, `Memory` and `Calldata`

Also, this tutorial is created to be easy to follow with, so you can also code yourself and practise.

# Commonly Asked Questions and Answers in Solidity

## 1. What is Solidity?

**Answer**: Solidity is a statically-typed programming language designed for writing smart contracts on the Ethereum blockchain platform.

---

## 2. What is the difference between `view`, `pure`, and `payable` function modifiers?

**Answer**: 
- `view`: This function will not modify state variables and is only used for viewing data.
- `pure`: Similar to `view`, but `pure` functions also can't read the state.
- `payable`: This function can receive Ether during its execution.

---

## 3. What is `msg.sender`?

**Answer**: `msg.sender` is a special variable that holds the address of the person or contract that initiated the current function call.

---

## 4. How do you import a library in Solidity?

**Answer**: Use the `import` statement to import a library. E.g., `import "@openzeppelin/contracts/token/ERC20/IERC20.sol";`.

---

## 5. What is the `storage` and `memory` data location?

**Answer**: 
- `storage`: Persistent storage on the blockchain, but changes are very costly in terms of gas.
- `memory`: Temporary storage, cheaper but only available during the execution of a function.

---

## 6. What are events and how are they used?

**Answer**: Events allow logging to the Ethereum blockchain. These logs can be easily accessed from a frontend application and are an effective way to "return" data from transactions, which otherwise don't yield a return value.

---

## 7. How do you write upgradable smart contracts?

**Answer**: Upgradable smart contracts often employ a proxy pattern. The logic and data are separated, and the logic can be changed without affecting the data.

---

## 8. What are mappings?

**Answer**: Mappings are hash tables that exist in the storage of a contract. They map keys to values, providing quick look-up capabilities.

---

## 9. What is the difference between `public` and `external` function visibility?

**Answer**: 
- `public`: Can be called both internally and externally.
- `external`: Can only be called externally, i.e., from outside the current contract.

---

## 10. How do you handle re-entrancy attacks?

**Answer**: To prevent re-entrancy attacks, you can use the Checks-Effects-Interactions pattern and make sure to not call external contracts until you've done all the internal work you need to do.

---

## 11. What is the gas limit and gas price?

**Answer**: 
- Gas limit: The maximum amount of gas the user is willing to spend on a transaction.
- Gas price: The amount of Ether the user is willing to spend per unit of gas.

---

## 12. How do you debug a Solidity smart contract?

**Answer**: You can debug using tools like Remix, Hardhat, or Truffle Debugger, which allow you to step through each line of your Solidity code.

---

## 13. How do I interact with a deployed contract?

**Answer**: You can interact with a deployed contract through Web3.js or Ethers.js libraries in a JavaScript environment, or directly through a Solidity interface in a contract-to-contract call.

---

## 14. What are Solidity modifiers?

**Answer**: Modifiers are function decorators that can be used to change the behavior of functions in a declarative way.

---

## 15. How do you manage different versions of Solidity?

**Answer**: You specify the compiler version with the `pragma solidity ^x.y.z;` statement. It’s also possible to specify a range of versions.

---

These are just a few of the commonly asked questions in the world of Solidity. I hope you find this helpful!

