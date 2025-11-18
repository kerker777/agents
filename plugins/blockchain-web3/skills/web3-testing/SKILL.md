---
name: web3-testing
description: 使用 Hardhat 和 Foundry 進行智能合約的全面測試，包括單元測試、整合測試和主網分叉。適用於測試 Solidity 合約、建立區塊鏈測試套件或驗證 DeFi 協議。
---

# Web3 智能合約測試

掌握使用 Hardhat、Foundry 和進階測試模式進行智能合約的全面測試策略。

## 使用此技能的時機

- 為智能合約撰寫單元測試
- 建立整合測試套件
- 執行 gas 優化測試
- 模糊測試以找出邊界案例
- 分叉主網進行真實測試
- 自動化測試覆蓋率報告
- 在 Etherscan 上驗證合約

## Hardhat 測試設定

```javascript
// hardhat.config.js
require("@nomicfoundation/hardhat-toolbox");
require("@nomiclabs/hardhat-etherscan");
require("hardhat-gas-reporter");
require("solidity-coverage");

module.exports = {
  solidity: {
    version: "0.8.19",
    settings: {
      optimizer: {
        enabled: true,
        runs: 200
      }
    }
  },
  networks: {
    hardhat: {
      forking: {
        url: process.env.MAINNET_RPC_URL,
        blockNumber: 15000000
      }
    },
    goerli: {
      url: process.env.GOERLI_RPC_URL,
      accounts: [process.env.PRIVATE_KEY]
    }
  },
  gasReporter: {
    enabled: true,
    currency: 'USD',
    coinmarketcap: process.env.COINMARKETCAP_API_KEY
  },
  etherscan: {
    apiKey: process.env.ETHERSCAN_API_KEY
  }
};
```

## 單元測試模式

```javascript
const { expect } = require("chai");
const { ethers } = require("hardhat");
const { loadFixture, time } = require("@nomicfoundation/hardhat-network-helpers");

describe("Token Contract", function () {
  // 測試設定的 Fixture
  async function deployTokenFixture() {
    const [owner, addr1, addr2] = await ethers.getSigners();

    const Token = await ethers.getContractFactory("Token");
    const token = await Token.deploy();

    return { token, owner, addr1, addr2 };
  }

  describe("Deployment", function () {
    it("Should set the right owner", async function () {
      const { token, owner } = await loadFixture(deployTokenFixture);
      expect(await token.owner()).to.equal(owner.address);
    });

    it("Should assign total supply to owner", async function () {
      const { token, owner } = await loadFixture(deployTokenFixture);
      const ownerBalance = await token.balanceOf(owner.address);
      expect(await token.totalSupply()).to.equal(ownerBalance);
    });
  });

  describe("Transactions", function () {
    it("Should transfer tokens between accounts", async function () {
      const { token, owner, addr1 } = await loadFixture(deployTokenFixture);

      await expect(token.transfer(addr1.address, 50))
        .to.changeTokenBalances(token, [owner, addr1], [-50, 50]);
    });

    it("Should fail if sender doesn't have enough tokens", async function () {
      const { token, addr1 } = await loadFixture(deployTokenFixture);
      const initialBalance = await token.balanceOf(addr1.address);

      await expect(
        token.connect(addr1).transfer(owner.address, 1)
      ).to.be.revertedWith("Insufficient balance");
    });

    it("Should emit Transfer event", async function () {
      const { token, owner, addr1 } = await loadFixture(deployTokenFixture);

      await expect(token.transfer(addr1.address, 50))
        .to.emit(token, "Transfer")
        .withArgs(owner.address, addr1.address, 50);
    });
  });

  describe("Time-based tests", function () {
    it("Should handle time-locked operations", async function () {
      const { token } = await loadFixture(deployTokenFixture);

      // 將時間增加 1 天
      await time.increase(86400);

      // 測試時間相依的功能
    });
  });

  describe("Gas optimization", function () {
    it("Should use gas efficiently", async function () {
      const { token } = await loadFixture(deployTokenFixture);

      const tx = await token.transfer(addr1.address, 100);
      const receipt = await tx.wait();

      expect(receipt.gasUsed).to.be.lessThan(50000);
    });
  });
});
```

## Foundry 測試 (Forge)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "forge-std/Test.sol";
import "../src/Token.sol";

contract TokenTest is Test {
    Token token;
    address owner = address(1);
    address user1 = address(2);
    address user2 = address(3);

    function setUp() public {
        vm.prank(owner);
        token = new Token();
    }

    function testInitialSupply() public {
        assertEq(token.totalSupply(), 1000000 * 10**18);
    }

    function testTransfer() public {
        vm.prank(owner);
        token.transfer(user1, 100);

        assertEq(token.balanceOf(user1), 100);
        assertEq(token.balanceOf(owner), token.totalSupply() - 100);
    }

    function testFailTransferInsufficientBalance() public {
        vm.prank(user1);
        token.transfer(user2, 100); // Should fail
    }

    function testCannotTransferToZeroAddress() public {
        vm.prank(owner);
        vm.expectRevert("Invalid recipient");
        token.transfer(address(0), 100);
    }

    // 模糊測試
    function testFuzzTransfer(uint256 amount) public {
        vm.assume(amount > 0 && amount <= token.totalSupply());

        vm.prank(owner);
        token.transfer(user1, amount);

        assertEq(token.balanceOf(user1), amount);
    }

    // 使用 cheatcodes 進行測試
    function testDealAndPrank() public {
        // 給予地址 ETH
        vm.deal(user1, 10 ether);

        // 模擬地址
        vm.prank(user1);

        // 測試功能
        assertEq(user1.balance, 10 ether);
    }

    // 主網分叉測試
    function testForkMainnet() public {
        vm.createSelectFork("https://eth-mainnet.alchemyapi.io/v2/...");

        // 與主網合約互動
        address dai = 0x6B175474E89094C44Da98b954EedeAC495271d0F;
        assertEq(IERC20(dai).symbol(), "DAI");
    }
}
```

## 進階測試模式

### 快照與還原
```javascript
describe("Complex State Changes", function () {
  let snapshotId;

  beforeEach(async function () {
    snapshotId = await network.provider.send("evm_snapshot");
  });

  afterEach(async function () {
    await network.provider.send("evm_revert", [snapshotId]);
  });

  it("Test 1", async function () {
    // 進行狀態變更
  });

  it("Test 2", async function () {
    // 狀態已還原，全新開始
  });
});
```

### 主網分叉
```javascript
describe("Mainnet Fork Tests", function () {
  let uniswapRouter, dai, usdc;

  before(async function () {
    await network.provider.request({
      method: "hardhat_reset",
      params: [{
        forking: {
          jsonRpcUrl: process.env.MAINNET_RPC_URL,
          blockNumber: 15000000
        }
      }]
    });

    // 連接到現有的主網合約
    uniswapRouter = await ethers.getContractAt(
      "IUniswapV2Router",
      "0x7a250d5630B4cF539739dF2C5dAcb4c659F2488D"
    );

    dai = await ethers.getContractAt(
      "IERC20",
      "0x6B175474E89094C44Da98b954EedeAC495271d0F"
    );
  });

  it("Should swap on Uniswap", async function () {
    // 使用真實的 Uniswap 合約進行測試
  });
});
```

### 模擬帳戶
```javascript
it("Should impersonate whale account", async function () {
  const whaleAddress = "0x...";

  await network.provider.request({
    method: "hardhat_impersonateAccount",
    params: [whaleAddress]
  });

  const whale = await ethers.getSigner(whaleAddress);

  // 使用巨鯨的代幣
  await dai.connect(whale).transfer(addr1.address, ethers.utils.parseEther("1000"));
});
```

## Gas 優化測試

```javascript
const { expect } = require("chai");

describe("Gas Optimization", function () {
  it("Compare gas usage between implementations", async function () {
    const Implementation1 = await ethers.getContractFactory("OptimizedContract");
    const Implementation2 = await ethers.getContractFactory("UnoptimizedContract");

    const contract1 = await Implementation1.deploy();
    const contract2 = await Implementation2.deploy();

    const tx1 = await contract1.doSomething();
    const receipt1 = await tx1.wait();

    const tx2 = await contract2.doSomething();
    const receipt2 = await tx2.wait();

    console.log("Optimized gas:", receipt1.gasUsed.toString());
    console.log("Unoptimized gas:", receipt2.gasUsed.toString());

    expect(receipt1.gasUsed).to.be.lessThan(receipt2.gasUsed);
  });
});
```

## 覆蓋率報告

```bash
# 產生覆蓋率報告
npx hardhat coverage

# 輸出顯示：
# File                | % Stmts | % Branch | % Funcs | % Lines |
# -------------------|---------|----------|---------|---------|
# contracts/Token.sol |   100   |   90     |   100   |   95    |
```

## 合約驗證

```javascript
// 在 Etherscan 上驗證
await hre.run("verify:verify", {
  address: contractAddress,
  constructorArguments: [arg1, arg2]
});
```

```bash
# 或透過 CLI
npx hardhat verify --network mainnet CONTRACT_ADDRESS "Constructor arg1" "arg2"
```

## CI/CD 整合

```yaml
# .github/workflows/test.yml
name: Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-node@v2
        with:
          node-version: '16'

      - run: npm install
      - run: npx hardhat compile
      - run: npx hardhat test
      - run: npx hardhat coverage

      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v2
```

## 資源

- **references/hardhat-setup.md**: Hardhat 配置指南
- **references/foundry-setup.md**: Foundry 測試框架
- **references/test-patterns.md**: 測試最佳實踐
- **references/mainnet-forking.md**: 分叉測試策略
- **references/contract-verification.md**: Etherscan 驗證
- **assets/hardhat-config.js**: 完整的 Hardhat 配置
- **assets/test-suite.js**: 綜合測試範例
- **assets/foundry.toml**: Foundry 配置
- **scripts/test-contract.sh**: 自動化測試腳本

## 最佳實踐

1. **測試覆蓋率**：目標達成 >90% 覆蓋率
2. **邊界案例**：測試邊界條件
3. **Gas 限制**：驗證函式不會達到區塊 gas 上限
4. **重入攻擊**：測試重入攻擊漏洞
5. **存取控制**：測試未經授權的存取嘗試
6. **事件**：驗證事件的觸發
7. **Fixtures**：使用 fixtures 避免程式碼重複
8. **主網分叉**：使用真實合約進行測試
9. **模糊測試**：使用基於屬性的測試
10. **CI/CD**：在每次提交時自動化測試
