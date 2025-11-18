---
name: solidity-security
description: 精通智能合約安全最佳實務，預防常見漏洞並實作安全的 Solidity 模式。適用於撰寫智能合約、審計既有合約，或為區塊鏈應用程式實作安全措施時使用。
---

# Solidity Security

精通智能合約安全最佳實務、漏洞預防，以及安全的 Solidity 開發模式。

## 何時使用此技能

- 撰寫安全的智能合約
- 審計既有合約的漏洞
- 實作安全的 DeFi 協議
- 預防重入、溢位及存取控制問題
- 在維持安全性的同時優化 gas 使用
- 準備合約進行專業審計
- 理解常見的攻擊向量

## 關鍵漏洞

### 1. 重入攻擊 (Reentrancy)
攻擊者在狀態更新前回呼您的合約。

**存在漏洞的程式碼：**
```solidity
// VULNERABLE TO REENTRANCY
contract VulnerableBank {
    mapping(address => uint256) public balances;

    function withdraw() public {
        uint256 amount = balances[msg.sender];

        // DANGER: External call before state update
        (bool success, ) = msg.sender.call{value: amount}("");
        require(success);

        balances[msg.sender] = 0;  // Too late!
    }
}
```

**安全模式 (Checks-Effects-Interactions)：**
```solidity
contract SecureBank {
    mapping(address => uint256) public balances;

    function withdraw() public {
        uint256 amount = balances[msg.sender];
        require(amount > 0, "Insufficient balance");

        // EFFECTS: Update state BEFORE external call
        balances[msg.sender] = 0;

        // INTERACTIONS: External call last
        (bool success, ) = msg.sender.call{value: amount}("");
        require(success, "Transfer failed");
    }
}
```

**替代方案：ReentrancyGuard**
```solidity
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";

contract SecureBank is ReentrancyGuard {
    mapping(address => uint256) public balances;

    function withdraw() public nonReentrant {
        uint256 amount = balances[msg.sender];
        require(amount > 0, "Insufficient balance");

        balances[msg.sender] = 0;

        (bool success, ) = msg.sender.call{value: amount}("");
        require(success, "Transfer failed");
    }
}
```

### 2. 整數溢位/下溢 (Integer Overflow/Underflow)

**存在漏洞的程式碼 (Solidity < 0.8.0)：**
```solidity
// VULNERABLE
contract VulnerableToken {
    mapping(address => uint256) public balances;

    function transfer(address to, uint256 amount) public {
        // No overflow check - can wrap around
        balances[msg.sender] -= amount;  // Can underflow!
        balances[to] += amount;          // Can overflow!
    }
}
```

**安全模式 (Solidity >= 0.8.0)：**
```solidity
// Solidity 0.8+ has built-in overflow/underflow checks
contract SecureToken {
    mapping(address => uint256) public balances;

    function transfer(address to, uint256 amount) public {
        // Automatically reverts on overflow/underflow
        balances[msg.sender] -= amount;
        balances[to] += amount;
    }
}
```

**對於 Solidity < 0.8.0，使用 SafeMath：**
```solidity
import "@openzeppelin/contracts/utils/math/SafeMath.sol";

contract SecureToken {
    using SafeMath for uint256;
    mapping(address => uint256) public balances;

    function transfer(address to, uint256 amount) public {
        balances[msg.sender] = balances[msg.sender].sub(amount);
        balances[to] = balances[to].add(amount);
    }
}
```

### 3. 存取控制 (Access Control)

**存在漏洞的程式碼：**
```solidity
// VULNERABLE: Anyone can call critical functions
contract VulnerableContract {
    address public owner;

    function withdraw(uint256 amount) public {
        // No access control!
        payable(msg.sender).transfer(amount);
    }
}
```

**安全模式：**
```solidity
import "@openzeppelin/contracts/access/Ownable.sol";

contract SecureContract is Ownable {
    function withdraw(uint256 amount) public onlyOwner {
        payable(owner()).transfer(amount);
    }
}

// Or implement custom role-based access
contract RoleBasedContract {
    mapping(address => bool) public admins;

    modifier onlyAdmin() {
        require(admins[msg.sender], "Not an admin");
        _;
    }

    function criticalFunction() public onlyAdmin {
        // Protected function
    }
}
```

### 4. 搶先交易 (Front-Running)

**存在漏洞：**
```solidity
// VULNERABLE TO FRONT-RUNNING
contract VulnerableDEX {
    function swap(uint256 amount, uint256 minOutput) public {
        // Attacker sees this in mempool and front-runs
        uint256 output = calculateOutput(amount);
        require(output >= minOutput, "Slippage too high");
        // Perform swap
    }
}
```

**緩解措施：**
```solidity
contract SecureDEX {
    mapping(bytes32 => bool) public usedCommitments;

    // Step 1: Commit to trade
    function commitTrade(bytes32 commitment) public {
        usedCommitments[commitment] = true;
    }

    // Step 2: Reveal trade (next block)
    function revealTrade(
        uint256 amount,
        uint256 minOutput,
        bytes32 secret
    ) public {
        bytes32 commitment = keccak256(abi.encodePacked(
            msg.sender, amount, minOutput, secret
        ));
        require(usedCommitments[commitment], "Invalid commitment");
        // Perform swap
    }
}
```

## 安全最佳實務

### Checks-Effects-Interactions 模式
```solidity
contract SecurePattern {
    mapping(address => uint256) public balances;

    function withdraw(uint256 amount) public {
        // 1. CHECKS: Validate conditions
        require(amount <= balances[msg.sender], "Insufficient balance");
        require(amount > 0, "Amount must be positive");

        // 2. EFFECTS: Update state
        balances[msg.sender] -= amount;

        // 3. INTERACTIONS: External calls last
        (bool success, ) = msg.sender.call{value: amount}("");
        require(success, "Transfer failed");
    }
}
```

### Pull Over Push 模式
```solidity
// Prefer this (pull)
contract SecurePayment {
    mapping(address => uint256) public pendingWithdrawals;

    function recordPayment(address recipient, uint256 amount) internal {
        pendingWithdrawals[recipient] += amount;
    }

    function withdraw() public {
        uint256 amount = pendingWithdrawals[msg.sender];
        require(amount > 0, "Nothing to withdraw");

        pendingWithdrawals[msg.sender] = 0;
        payable(msg.sender).transfer(amount);
    }
}

// Over this (push)
contract RiskyPayment {
    function distributePayments(address[] memory recipients, uint256[] memory amounts) public {
        for (uint i = 0; i < recipients.length; i++) {
            // If any transfer fails, entire batch fails
            payable(recipients[i]).transfer(amounts[i]);
        }
    }
}
```

### 輸入驗證
```solidity
contract SecureContract {
    function transfer(address to, uint256 amount) public {
        // Validate inputs
        require(to != address(0), "Invalid recipient");
        require(to != address(this), "Cannot send to contract");
        require(amount > 0, "Amount must be positive");
        require(amount <= balances[msg.sender], "Insufficient balance");

        // Proceed with transfer
        balances[msg.sender] -= amount;
        balances[to] += amount;
    }
}
```

### 緊急停止 (Circuit Breaker)
```solidity
import "@openzeppelin/contracts/security/Pausable.sol";

contract EmergencyStop is Pausable, Ownable {
    function criticalFunction() public whenNotPaused {
        // Function logic
    }

    function emergencyStop() public onlyOwner {
        _pause();
    }

    function resume() public onlyOwner {
        _unpause();
    }
}
```

## Gas 優化

### 使用 `uint256` 而非較小的型別
```solidity
// More gas efficient
contract GasEfficient {
    uint256 public value;  // Optimal

    function set(uint256 _value) public {
        value = _value;
    }
}

// Less efficient
contract GasInefficient {
    uint8 public value;  // Still uses 256-bit slot

    function set(uint8 _value) public {
        value = _value;  // Extra gas for type conversion
    }
}
```

### 打包儲存變數
```solidity
// Gas efficient (3 variables in 1 slot)
contract PackedStorage {
    uint128 public a;  // Slot 0
    uint64 public b;   // Slot 0
    uint64 public c;   // Slot 0
    uint256 public d;  // Slot 1
}

// Gas inefficient (each variable in separate slot)
contract UnpackedStorage {
    uint256 public a;  // Slot 0
    uint256 public b;  // Slot 1
    uint256 public c;  // Slot 2
    uint256 public d;  // Slot 3
}
```

### 對函式參數使用 `calldata` 而非 `memory`
```solidity
contract GasOptimized {
    // More gas efficient
    function processData(uint256[] calldata data) public pure returns (uint256) {
        return data[0];
    }

    // Less efficient
    function processDataMemory(uint256[] memory data) public pure returns (uint256) {
        return data[0];
    }
}
```

### 使用事件來儲存資料 (適當情況下)
```solidity
contract EventStorage {
    // Emitting events is cheaper than storage
    event DataStored(address indexed user, uint256 indexed id, bytes data);

    function storeData(uint256 id, bytes calldata data) public {
        emit DataStored(msg.sender, id, data);
        // Don't store in contract storage unless needed
    }
}
```

## 常見漏洞檢查清單

```solidity
// Security Checklist Contract
contract SecurityChecklist {
    /**
     * [ ] 重入保護 (ReentrancyGuard 或 CEI 模式)
     * [ ] 整數溢位/下溢 (Solidity 0.8+ 或 SafeMath)
     * [ ] 存取控制 (Ownable, roles, modifiers)
     * [ ] 輸入驗證 (require 陳述式)
     * [ ] 搶先交易緩解 (適用時使用 commit-reveal)
     * [ ] Gas 優化 (打包儲存, calldata)
     * [ ] 緊急停止機制 (Pausable)
     * [ ] 支付使用 pull over push 模式
     * [ ] 不對不受信任的合約使用 delegatecall
     * [ ] 不使用 tx.origin 進行身份驗證 (使用 msg.sender)
     * [ ] 適當的事件發出
     * [ ] 外部呼叫放在函式最後
     * [ ] 檢查外部呼叫的回傳值
     * [ ] 不使用硬編碼的地址
     * [ ] 升級機制 (如果使用代理模式)
     */
}
```

## 安全測試

```javascript
// Hardhat test example
const { expect } = require("chai");
const { ethers } = require("hardhat");

describe("Security Tests", function () {
    it("Should prevent reentrancy attack", async function () {
        const [attacker] = await ethers.getSigners();

        const VictimBank = await ethers.getContractFactory("SecureBank");
        const bank = await VictimBank.deploy();

        const Attacker = await ethers.getContractFactory("ReentrancyAttacker");
        const attackerContract = await Attacker.deploy(bank.address);

        // Deposit funds
        await bank.deposit({value: ethers.utils.parseEther("10")});

        // Attempt reentrancy attack
        await expect(
            attackerContract.attack({value: ethers.utils.parseEther("1")})
        ).to.be.revertedWith("ReentrancyGuard: reentrant call");
    });

    it("Should prevent integer overflow", async function () {
        const Token = await ethers.getContractFactory("SecureToken");
        const token = await Token.deploy();

        // Attempt overflow
        await expect(
            token.transfer(attacker.address, ethers.constants.MaxUint256)
        ).to.be.reverted;
    });

    it("Should enforce access control", async function () {
        const [owner, attacker] = await ethers.getSigners();

        const Contract = await ethers.getContractFactory("SecureContract");
        const contract = await Contract.deploy();

        // Attempt unauthorized withdrawal
        await expect(
            contract.connect(attacker).withdraw(100)
        ).to.be.revertedWith("Ownable: caller is not the owner");
    });
});
```

## 審計準備

```solidity
contract WellDocumentedContract {
    /**
     * @title Well Documented Contract
     * @dev Example of proper documentation for audits
     * @notice This contract handles user deposits and withdrawals
     */

    /// @notice Mapping of user balances
    mapping(address => uint256) public balances;

    /**
     * @dev Deposits ETH into the contract
     * @notice Anyone can deposit funds
     */
    function deposit() public payable {
        require(msg.value > 0, "Must send ETH");
        balances[msg.sender] += msg.value;
    }

    /**
     * @dev Withdraws user's balance
     * @notice Follows CEI pattern to prevent reentrancy
     * @param amount Amount to withdraw in wei
     */
    function withdraw(uint256 amount) public {
        // CHECKS
        require(amount <= balances[msg.sender], "Insufficient balance");

        // EFFECTS
        balances[msg.sender] -= amount;

        // INTERACTIONS
        (bool success, ) = msg.sender.call{value: amount}("");
        require(success, "Transfer failed");
    }
}
```

## 資源

- **references/reentrancy.md**: 全面的重入預防
- **references/access-control.md**: 基於角色的存取模式
- **references/overflow-underflow.md**: SafeMath 與整數安全
- **references/gas-optimization.md**: Gas 節省技巧
- **references/vulnerability-patterns.md**: 常見漏洞目錄
- **assets/solidity-contracts-templates.sol**: 安全的合約範本
- **assets/security-checklist.md**: 審計前檢查清單
- **scripts/analyze-contract.sh**: 靜態分析工具

## 安全分析工具

- **Slither**: 靜態分析工具
- **Mythril**: 安全分析工具
- **Echidna**: 模糊測試工具
- **Manticore**: 符號執行
- **Securify**: 自動化安全掃描器

## 常見陷阱

1. **使用 `tx.origin` 進行身份驗證**：應改用 `msg.sender`
2. **未檢查的外部呼叫**：務必檢查回傳值
3. **對不受信任的合約使用 Delegatecall**：可能劫持您的合約
4. **浮動的 Pragma**：應固定於特定的 Solidity 版本
5. **缺少事件**：對狀態變更應發出事件
6. **迴圈中過多的 Gas**：可能達到區塊 gas 限制
7. **沒有升級路徑**：如需升級請考慮使用代理模式
