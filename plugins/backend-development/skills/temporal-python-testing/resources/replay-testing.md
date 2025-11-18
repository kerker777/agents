# 重播測試用於確定性和相容性

使用重播測試驗證工作流程確定性並確保程式碼變更安全的完整指南。

## 什麼是重播測試？

**目的**：驗證工作流程程式碼變更與現有工作流程執行的向後相容性

**運作方式**：
1. Temporal 將每個工作流程決策記錄為事件歷史
2. 重播測試針對記錄的歷史重新執行工作流程程式碼
3. 如果新程式碼做出相同決策 → 具確定性（可安全部署）
4. 如果決策不同 → 不具確定性（重大變更）

**關鍵使用情境**：
- 部署工作流程程式碼變更到生產環境
- 驗證重構不會中斷執行中的工作流程
- CI/CD 自動化相容性檢查
- 版本遷移驗證

## 基本重播測試

### 重播器設定

```python
from temporalio.worker import Replayer
from temporalio.client import Client

async def test_workflow_replay():
    """針對生產環境歷史測試工作流程"""

    # 連接到 Temporal 伺服器
    client = await Client.connect("localhost:7233")

    # 使用當前工作流程程式碼建立重播器
    replayer = Replayer(
        workflows=[OrderWorkflow, PaymentWorkflow]
    )

    # 從生產環境取得工作流程歷史
    handle = client.get_workflow_handle("order-123")
    history = await handle.fetch_history()

    # 使用當前程式碼重播歷史
    await replayer.replay_workflow(history)
    # 成功 = 具確定性，例外 = 重大變更
```

### 針對多個歷史進行測試

```python
import pytest
from temporalio.worker import Replayer

@pytest.mark.asyncio
async def test_replay_multiple_workflows():
    """針對多個生產環境歷史進行重播"""

    replayer = Replayer(workflows=[OrderWorkflow])

    # 針對不同的工作流程執行進行測試
    workflow_ids = [
        "order-success-123",
        "order-cancelled-456",
        "order-retry-789",
    ]

    for workflow_id in workflow_ids:
        handle = client.get_workflow_handle(workflow_id)
        history = await handle.fetch_history()

        # 重播對所有變體都應成功
        await replayer.replay_workflow(history)
```

## 確定性驗證

### 常見的非確定性模式

**問題：隨機數生成**
```python
# ❌ 非確定性（中斷重播）
@workflow.defn
class BadWorkflow:
    @workflow.run
    async def run(self) -> int:
        return random.randint(1, 100)  # 重播時會不同！

# ✅ 確定性（重播安全）
@workflow.defn
class GoodWorkflow:
    @workflow.run
    async def run(self) -> int:
        return workflow.random().randint(1, 100)  # 確定性隨機
```

**問題：當前時間**
```python
# ❌ 非確定性
@workflow.defn
class BadWorkflow:
    @workflow.run
    async def run(self) -> str:
        now = datetime.now()  # 重播時會不同！
        return now.isoformat()

# ✅ 確定性
@workflow.defn
class GoodWorkflow:
    @workflow.run
    async def run(self) -> str:
        now = workflow.now()  # 確定性時間
        return now.isoformat()
```

**問題：直接外部呼叫**
```python
# ❌ 非確定性
@workflow.defn
class BadWorkflow:
    @workflow.run
    async def run(self) -> dict:
        response = requests.get("https://api.example.com/data")  # 外部呼叫！
        return response.json()

# ✅ 確定性
@workflow.defn
class GoodWorkflow:
    @workflow.run
    async def run(self) -> dict:
        # 使用活動進行外部呼叫
        return await workflow.execute_activity(
            fetch_external_data,
            start_to_close_timeout=timedelta(seconds=30),
        )
```

### 測試確定性

```python
@pytest.mark.asyncio
async def test_workflow_determinism():
    """驗證工作流程在多次執行時產生相同輸出"""

    @workflow.defn
    class DeterministicWorkflow:
        @workflow.run
        async def run(self, seed: int) -> list[int]:
            # 使用 workflow.random() 確保確定性
            rng = workflow.random()
            rng.seed(seed)
            return [rng.randint(1, 100) for _ in range(10)]

    env = await WorkflowEnvironment.start_time_skipping()

    # 使用相同輸入執行工作流程兩次
    results = []
    for i in range(2):
        async with Worker(
            env.client,
            task_queue="test",
            workflows=[DeterministicWorkflow],
        ):
            result = await env.client.execute_workflow(
                DeterministicWorkflow.run,
                42,  # 相同的種子
                id=f"determinism-test-{i}",
                task_queue="test",
            )
            results.append(result)

    await env.shutdown()

    # 驗證輸出相同
    assert results[0] == results[1]
```

## 生產環境歷史重播

### 匯出工作流程歷史

```python
from temporalio.client import Client

async def export_workflow_history(workflow_id: str, output_file: str):
    """匯出工作流程歷史用於重播測試"""

    client = await Client.connect("production.temporal.io:7233")

    # 取得工作流程歷史
    handle = client.get_workflow_handle(workflow_id)
    history = await handle.fetch_history()

    # 儲存到檔案用於重播測試
    with open(output_file, "wb") as f:
        f.write(history.SerializeToString())

    print(f"Exported history to {output_file}")
```

### 從檔案重播

```python
from temporalio.worker import Replayer
from temporalio.api.history.v1 import History

async def test_replay_from_file():
    """從匯出的歷史檔案重播工作流程"""

    # 從檔案載入歷史
    with open("workflow_histories/order-123.pb", "rb") as f:
        history = History.FromString(f.read())

    # 使用當前工作流程程式碼重播
    replayer = Replayer(workflows=[OrderWorkflow])
    await replayer.replay_workflow(history)
    # 成功 = 可安全部署
```

## CI/CD 整合模式

### GitHub Actions 範例

```yaml
# .github/workflows/replay-tests.yml
name: Replay Tests

on:
  pull_request:
    branches: [main]

jobs:
  replay-tests:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: "3.11"

      - name: Install dependencies
        run: |
          pip install -r requirements.txt
          pip install pytest pytest-asyncio

      - name: Download production histories
        run: |
          # Fetch recent workflow histories from production
          python scripts/export_histories.py

      - name: Run replay tests
        run: |
          pytest tests/replay/ --verbose

      - name: Upload results
        if: failure()
        uses: actions/upload-artifact@v3
        with:
          name: replay-failures
          path: replay-failures/
```

### 自動化歷史匯出

```python
# scripts/export_histories.py
import asyncio
from temporalio.client import Client
from datetime import datetime, timedelta

async def export_recent_histories():
    """匯出最近的生產環境工作流程歷史"""

    client = await Client.connect("production.temporal.io:7233")

    # 查詢最近完成的工作流程
    workflows = client.list_workflows(
        query="WorkflowType='OrderWorkflow' AND CloseTime > '7 days ago'"
    )

    count = 0
    async for workflow in workflows:
        # 匯出歷史
        history = await workflow.fetch_history()

        # 儲存到檔案
        filename = f"workflow_histories/{workflow.id}.pb"
        with open(filename, "wb") as f:
            f.write(history.SerializeToString())

        count += 1
        if count >= 100:  # 限制為最近 100 個
            break

    print(f"Exported {count} workflow histories")

if __name__ == "__main__":
    asyncio.run(export_recent_histories())
```

### 重播測試套件

```python
# tests/replay/test_workflow_replay.py
import pytest
import glob
from temporalio.worker import Replayer
from temporalio.api.history.v1 import History
from workflows import OrderWorkflow, PaymentWorkflow

@pytest.mark.asyncio
async def test_replay_all_histories():
    """重播所有生產環境歷史"""

    replayer = Replayer(
        workflows=[OrderWorkflow, PaymentWorkflow]
    )

    # 載入所有歷史檔案
    history_files = glob.glob("workflow_histories/*.pb")

    failures = []
    for history_file in history_files:
        try:
            with open(history_file, "rb") as f:
                history = History.FromString(f.read())

            await replayer.replay_workflow(history)
            print(f"✓ {history_file}")

        except Exception as e:
            failures.append((history_file, str(e)))
            print(f"✗ {history_file}: {e}")

    # 回報失敗
    if failures:
        pytest.fail(
            f"Replay failed for {len(failures)} workflows:\n"
            + "\n".join(f"  {file}: {error}" for file, error in failures)
        )
```

## 版本相容性測試

### 測試程式碼演進

```python
@pytest.mark.asyncio
async def test_workflow_version_compatibility():
    """測試帶有版本變更的工作流程"""

    @workflow.defn
    class EvolvingWorkflow:
        @workflow.run
        async def run(self) -> str:
            # 使用版本控制進行安全的程式碼演進
            version = workflow.get_version("feature-flag", 1, 2)

            if version == 1:
                # 舊行為
                return "version-1"
            else:
                # 新行為
                return "version-2"

    env = await WorkflowEnvironment.start_time_skipping()

    # 測試版本 1 行為
    async with Worker(
        env.client,
        task_queue="test",
        workflows=[EvolvingWorkflow],
    ):
        result_v1 = await env.client.execute_workflow(
            EvolvingWorkflow.run,
            id="evolving-v1",
            task_queue="test",
        )
        assert result_v1 == "version-1"

        # 模擬工作流程使用版本 2 再次執行
        result_v2 = await env.client.execute_workflow(
            EvolvingWorkflow.run,
            id="evolving-v2",
            task_queue="test",
        )
        # 新工作流程使用版本 2
        assert result_v2 == "version-2"

    await env.shutdown()
```

### 遷移策略

```python
# 階段 1：新增版本檢查
@workflow.defn
class MigratingWorkflow:
    @workflow.run
    async def run(self) -> dict:
        version = workflow.get_version("new-logic", 1, 2)

        if version == 1:
            # 舊邏輯（現有工作流程）
            return await self._old_implementation()
        else:
            # 新邏輯（新工作流程）
            return await self._new_implementation()

# 階段 2：所有舊工作流程完成後，移除舊程式碼
@workflow.defn
class MigratedWorkflow:
    @workflow.run
    async def run(self) -> dict:
        # 僅保留新邏輯
        return await self._new_implementation()
```

## 最佳實務

1. **部署前重播**：在部署工作流程變更之前，務必執行重播測試
2. **定期匯出**：持續匯出生產環境歷史用於測試
3. **CI/CD 整合**：在拉取請求檢查中進行自動化重播測試
4. **版本追蹤**：使用 workflow.get_version() 進行安全的程式碼演進
5. **歷史保留**：保留具代表性的工作流程歷史用於回歸測試
6. **確定性**：絕不使用 random()、datetime.now() 或直接外部呼叫
7. **全面測試**：針對各種工作流程執行路徑進行測試

## 常見重播錯誤

**非確定性錯誤**：
```
WorkflowNonDeterministicError: Workflow command mismatch at position 5
Expected: ScheduleActivityTask(activity_id='activity-1')
Got: ScheduleActivityTask(activity_id='activity-2')
```

**解決方案**：程式碼變更改變了工作流程決策序列

**版本不符錯誤**：
```
WorkflowVersionError: Workflow version changed from 1 to 2 without using get_version()
```

**解決方案**：使用 workflow.get_version() 進行向後相容的變更

## 其他資源

- Replay Testing: docs.temporal.io/develop/python/testing-suite#replay-testing
- Workflow Versioning: docs.temporal.io/workflows#versioning
- Determinism Guide: docs.temporal.io/workflows#deterministic-constraints
- CI/CD Integration: github.com/temporalio/samples-python/tree/main/.github/workflows
