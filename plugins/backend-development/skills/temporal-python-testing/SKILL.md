---
name: temporal-python-testing
description: 使用 pytest、時間跳躍和模擬策略測試 Temporal 工作流程。涵蓋單元測試、整合測試、重放測試和本地開發環境設定。適用於實作 Temporal 工作流程測試或除錯測試失敗問題。
---

# Temporal Python 測試策略

使用 pytest 進行 Temporal 工作流程的全面測試方法，提供針對特定測試場景的漸進式資源揭露。

## 何時使用此技能

- **單元測試工作流程** - 使用時間跳躍進行快速測試
- **整合測試** - 使用模擬活動測試工作流程
- **重放測試** - 根據正式環境歷史記錄驗證確定性
- **本地開發** - 設定 Temporal 伺服器和 pytest
- **CI/CD 整合** - 自動化測試流程
- **覆蓋率策略** - 達成 ≥80% 測試覆蓋率

## 測試理念

**建議方法**（來源：docs.temporal.io/develop/python/testing-suite）：
- 主要撰寫整合測試
- 使用 pytest 搭配非同步 fixtures
- 時間跳躍可實現快速回饋（長達一個月的工作流程 → 數秒完成）
- 模擬活動以隔離工作流程邏輯
- 使用重放測試驗證確定性

**三種測試類型**：
1. **單元測試**：使用時間跳躍測試工作流程，使用 ActivityEnvironment 測試活動
2. **整合測試**：使用模擬活動的 Workers
3. **端對端測試**：使用真實活動的完整 Temporal 伺服器（謹慎使用）

## 可用資源

此技能透過漸進式揭露提供詳細指引。根據您的測試需求載入特定資源：

### 單元測試資源
**檔案**：`resources/unit-testing.md`
**何時載入**：獨立測試個別工作流程或活動
**包含內容**：
- 具備時間跳躍功能的 WorkflowEnvironment
- 用於活動測試的 ActivityEnvironment
- 快速執行長時間運行的工作流程
- 手動時間推進模式
- pytest fixtures 和模式

### 整合測試資源
**檔案**：`resources/integration-testing.md`
**何時載入**：使用模擬外部相依性測試工作流程
**包含內容**：
- 活動模擬策略
- 錯誤注入模式
- 多活動工作流程測試
- Signal 和 Query 測試
- 覆蓋率策略

### 重放測試資源
**檔案**：`resources/replay-testing.md`
**何時載入**：驗證確定性或部署工作流程變更
**包含內容**：
- 確定性驗證
- 正式環境歷史記錄重放
- CI/CD 整合模式
- 版本相容性測試

### 本地開發資源
**檔案**：`resources/local-setup.md`
**何時載入**：設定開發環境
**包含內容**：
- Docker Compose 配置
- pytest 設定和配置
- 覆蓋率工具整合
- 開發工作流程

## 快速入門指南

### 基本工作流程測試

```python
import pytest
from temporalio.testing import WorkflowEnvironment
from temporalio.worker import Worker

@pytest.fixture
async def workflow_env():
    env = await WorkflowEnvironment.start_time_skipping()
    yield env
    await env.shutdown()

@pytest.mark.asyncio
async def test_workflow(workflow_env):
    async with Worker(
        workflow_env.client,
        task_queue="test-queue",
        workflows=[YourWorkflow],
        activities=[your_activity],
    ):
        result = await workflow_env.client.execute_workflow(
            YourWorkflow.run,
            args,
            id="test-wf-id",
            task_queue="test-queue",
        )
        assert result == expected
```

### 基本活動測試

```python
from temporalio.testing import ActivityEnvironment

async def test_activity():
    env = ActivityEnvironment()
    result = await env.run(your_activity, "test-input")
    assert result == expected_output
```

## 覆蓋率目標

**建議覆蓋率**（來源：docs.temporal.io 最佳實務）：
- **工作流程**：≥80% 邏輯覆蓋率
- **活動**：≥80% 邏輯覆蓋率
- **整合測試**：使用模擬活動覆蓋關鍵路徑
- **重放測試**：部署前測試所有工作流程版本

## 關鍵測試原則

1. **時間跳躍** - 長達一個月的工作流程可在數秒內完成測試
2. **模擬活動** - 將工作流程邏輯與外部相依性隔離
3. **重放測試** - 部署前驗證確定性
4. **高覆蓋率** - 正式環境工作流程目標 ≥80%
5. **快速回饋** - 單元測試在毫秒內完成

## 如何使用資源

**依需求載入特定資源**：
- 「顯示單元測試模式」→ 載入 `resources/unit-testing.md`
- 「如何模擬活動？」→ 載入 `resources/integration-testing.md`
- 「設定本地 Temporal 伺服器」→ 載入 `resources/local-setup.md`
- 「驗證確定性」→ 載入 `resources/replay-testing.md`

## 其他參考資料

- Python SDK Testing: docs.temporal.io/develop/python/testing-suite
- Testing Patterns: github.com/temporalio/temporal/blob/main/docs/development/testing.md
- Python Samples: github.com/temporalio/samples-python
