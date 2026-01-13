# Professional Smart Contract Development: End-to-End Guide
*A comprehensive guide for Software Engineering students, covering smart contract architecture, security, and lifecycle.*

---

## 1. Core Principles: The EVM & Infrastructure
Before writing code, it's essential to understand the environment where it executes.
- **EVM (Ethereum Virtual Machine)**: A global, decentralized computer. Contracts are compiled into **Bytecode** that the EVM executes.
- **Gas**: The fuel for transactions. Every operation (adding, writing to storage) costs gas. 
  - **Writing to storage** is the most expensive (~20,000 gas for new slots).
  - **Updating storage** is cheaper (~5,000 gas).
- **Nodes & Providers**: To interact with the blockchain, you need an Ethereum node (e.g., Geth). Tools like **MetaMask** connect to nodes via **Infura** using **JSON-RPC** calls.

---

## 2. Anatomy & Skeleton of a Contract
A Solidity file (`.sol`) follows a strict structure.

```solidity
// 1. SPDX License Identifier (e.g., // SPDX-License-Identifier: MIT)
// 2. Pragmas (Compiler version: pragma solidity ^0.8.0;)

contract MyContract {
    // 3. State Variables (stored in Storage)
    // 4. Events (for off-chain logging)
    // 5. Modifiers (reusable logic)
    // 6. Constructor (runs once on deployment)
    // 7. Functions (external/public/internal/private)
}
```

### Data Locations (CRITICAL for Gas)
1.  **Storage**: Permanent data on the blockchain. Expensive.
2.  **Memory**: Temporary data wiped after function execution. Used for return values.
3.  **Calldata**: Read-only, non-persistent space for function arguments. **Best practice for saving gas** in external functions.

---

## 3. Advanced Data Structuring
Real-world apps use `structs` and `mappings` to manage complexity efficiently.

### Example: Social Network Architecture (`Exercises/13-Twitter-Struct-Exercise.sol`)
```solidity
contract Twitter {
    struct Tweet {
        uint256 id;
        address author;
        string content;
        uint256 timestamp;
        uint256 likes;
    }

    // Mapping: O(1) lookup. Maps User Address -> List of their Tweets
    mapping(address => Tweet[]) public tweets;

    // Constant (compile-time) & Immutable (deploy-time) save gas!
    uint16 public constant MAX_TWEET_LENGTH = 280; 
}
```

---

## 4. Execution Logic: Functions & Visibility
### Visibility Levels
- **External**: Only callable from outside. Most gas-efficient for receiving data.
- **Public**: Callable inside and outside.
- **Internal**: Callable only inside and by derived (child) contracts.
- **Private**: Callable only within this specific contract.

### Function Types
- **View**: Reads state but doesn't modify it (Free if called externally).
- **Pure**: Doesn't read or modify state (Free).
- **Payable**: Necessary for any function intended to receive **Ether** (`msg.value`).

---

## 5. Robust Error Handling
Handling failures correctly prevents lost funds and "bricked" contracts.

| Function | Gas Behavior | Primary Use Case |
| :--- | :--- | :--- |
| `require(cond, msg)` | Returns remaining gas | **Validating User Inputs** ("You did something wrong"). |
| `assert(cond)` | **Burns all gas** | **Internal Invariants** ("The contract has a bug"). |
| `revert(msg)` | Returns remaining gas | Immediate failure in complex logic. |

---

## 6. Communication: Events & The Gas "Anti-Pattern"
Blockchains are poor at "searching" data. Never loop over a dynamic array to find something (the **Gas Anti-Pattern**); the transaction will eventually exceed the `Block Gas Limit` and fail forever.

**Solution: Use Events!**
- Events log data in a special "Transaction Log" area.
- Off-chain apps (Websites) listen to these logs.
- Much cheaper than storing data in `Storage`.
- Use `indexed` parameters (up to 3) to allow efficient filtering (e.g., filter by `author`).

---

## 7. Architectural Patterns
### Inheritance
Solidity supports OOP. Use `is` to inherit, `virtual` for parents, and `override` for children.
```solidity
contract Base { function act() public virtual { ... } }
contract Derived is Base { function act() public override { super.act(); } }
```

### Interaction & Oracles
- **Interfaces**: Define the "shape" of another contract. Required for interaction.
- **Oracles (e.g., Chainlink)**: Since the EVM is isolated, it cannot see the outside world. Oracles provide off-chain data (price feeds, weather, random numbers) via callback functions.

### Upgradeability (The Proxy Pattern)
Smart contracts are immutable by default, but you can use **Proxies** to fix bugs:
1.  **Proxy Contract**: Holds the data and the address.
2.  **Implementation Contract**: Holds the logic.
3.  **`delegatecall`**: The Proxy calls the logic contract, but the logic executes **using the Proxy's storage**. To upgrade, the Proxy admin simply points to a new Implementation address.

---

## 8. Smart Contract Security (High Stakes)
Smart contracts exist in an adversarial environment. Bugs = Direct Financial Loss.

### 🛡️ Reentrancy (The DAO Hack)
The most famous exploit. An attacker calls a function, which calls the attacker's contract, which calls the original function again *before* the first call finishes.
- **Fix 1: Checks-Effects-Interactions**: Update your state (balances = 0) **before** sending Ether.
- **Fix 2: Reentrancy Guard**: Use a `nonReentrant` modifier.

### 🛡️ Frontrunning
Transaction pools are public. Attackers can see your pending transaction and send their own with a higher **Gas Price** to get processed first.
- **Countermeasure**: **Commit-Reveal Schemes**. Commit a hash of your choice first, then reveal the choice in a later block.

### 🛡️ Flash Loans
Uncollateralized loans that must be repaid in the same transaction. Powerful for arbitrage, but dangerous if used to manipulate governance or price oracles.

---

## 9. The Professional Workflow
1.  **Develop**: Write code in **Remix IDE** or Hardhat.
2.  **Compile**: Generate **ABI** (the JSON interface) and **Bytecode**.
3.  **Test**: Deploy to **Remix VM** (Local) then **Sepolia Testnet**.
4.  **Deploy**: Use **MetaMask** to push to Mainnet.
5.  **Verify**: Visit **Etherscan** and upload your source code. Verification matches your code to the on-chain bytecode, ensuring transparency and trust.

---

## Useful Links
- [Solidity by Example](https://solidity-by-example.org/)
- [OpenZeppelin Contracts](https://docs.openzeppelin.com/) (Industry Standard Libraries)
- [CryptoZombies](https://cryptozombies.io/) (Interactive Learning)

