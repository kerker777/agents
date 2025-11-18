# 使用模擬 Activity 的整合測試

全面涵蓋使用模擬外部相依性、錯誤注入及複雜場景來測試工作流程的模式。

## Activity 模擬策略

**目的**：在不呼叫真實外部服務的情況下測試工作流程編排邏輯

### 基本模擬模式

```python
import pytest
from temporalio.testing import WorkflowEnvironment
from temporalio.worker import Worker
from unittest.mock import Mock

@pytest.mark.asyncio
async def test_workflow_with_mocked_activity(workflow_env):
    """模擬 activity 以測試工作流程邏輯"""

    # 建立模擬 activity
    mock_activity = Mock(return_value="mocked-result")

    @workflow.defn
    class WorkflowWithActivity:
        @workflow.run
        async def run(self, input: str) -> str:
            result = await workflow.execute_activity(
                process_external_data,
                input,
                start_to_close_timeout=timedelta(seconds=10),
            )
            return f"processed: {result}"

    async with Worker(
        workflow_env.client,
        task_queue="test",
        workflows=[WorkflowWithActivity],
        activities=[mock_activity],  # 使用模擬而非真實 activity
    ):
        result = await workflow_env.client.execute_workflow(
            WorkflowWithActivity.run,
            "test-input",
            id="wf-mock",
            task_queue="test",
        )
        assert result == "processed: mocked-result"
        mock_activity.assert_called_once()
```

### 動態模擬回應

**基於場景的模擬**：
```python
@pytest.mark.asyncio
async def test_workflow_multiple_mock_scenarios(workflow_env):
    """使用動態模擬測試不同的工作流程路徑"""

    # 模擬根據輸入回傳不同值
    def dynamic_activity(input: str) -> str:
        if input == "error-case":
            raise ApplicationError("Validation failed", non_retryable=True)
        return f"processed-{input}"

    @workflow.defn
    class DynamicWorkflow:
        @workflow.run
        async def run(self, input: str) -> str:
            try:
                result = await workflow.execute_activity(
                    dynamic_activity,
                    input,
                    start_to_close_timeout=timedelta(seconds=10),
                )
                return f"success: {result}"
            except ApplicationError as e:
                return f"error: {e.message}"

    async with Worker(
        workflow_env.client,
        task_queue="test",
        workflows=[DynamicWorkflow],
        activities=[dynamic_activity],
    ):
        # 測試成功路徑
        result_success = await workflow_env.client.execute_workflow(
            DynamicWorkflow.run,
            "valid-input",
            id="wf-success",
            task_queue="test",
        )
        assert result_success == "success: processed-valid-input"

        # 測試錯誤路徑
        result_error = await workflow_env.client.execute_workflow(
            DynamicWorkflow.run,
            "error-case",
            id="wf-error",
            task_queue="test",
        )
        assert "Validation failed" in result_error
```

## 錯誤注入模式

### 測試暫時性失敗

**重試行為**：
```python
@pytest.mark.asyncio
async def test_workflow_transient_errors(workflow_env):
    """使用可控的失敗測試重試邏輯"""

    attempt_count = 0

    @activity.defn
    async def transient_activity() -> str:
        nonlocal attempt_count
        attempt_count += 1

        if attempt_count < 3:
            raise Exception(f"Transient error {attempt_count}")
        return "success-after-retries"

    @workflow.defn
    class RetryWorkflow:
        @workflow.run
        async def run(self) -> str:
            return await workflow.execute_activity(
                transient_activity,
                start_to_close_timeout=timedelta(seconds=10),
                retry_policy=RetryPolicy(
                    initial_interval=timedelta(milliseconds=10),
                    maximum_attempts=5,
                    backoff_coefficient=1.0,
                ),
            )

    async with Worker(
        workflow_env.client,
        task_queue="test",
        workflows=[RetryWorkflow],
        activities=[transient_activity],
    ):
        result = await workflow_env.client.execute_workflow(
            RetryWorkflow.run,
            id="retry-wf",
            task_queue="test",
        )
        assert result == "success-after-retries"
        assert attempt_count == 3
```

### 測試不可重試錯誤

**業務驗證失敗**：
```python
@pytest.mark.asyncio
async def test_workflow_non_retryable_error(workflow_env):
    """測試永久性失敗的處理"""

    @activity.defn
    async def validation_activity(input: dict) -> str:
        if not input.get("valid"):
            raise ApplicationError(
                "Invalid input",
                non_retryable=True,  # 不要重試驗證錯誤
            )
        return "validated"

    @workflow.defn
    class ValidationWorkflow:
        @workflow.run
        async def run(self, input: dict) -> str:
            try:
                return await workflow.execute_activity(
                    validation_activity,
                    input,
                    start_to_close_timeout=timedelta(seconds=10),
                )
            except ApplicationError as e:
                return f"validation-failed: {e.message}"

    async with Worker(
        workflow_env.client,
        task_queue="test",
        workflows=[ValidationWorkflow],
        activities=[validation_activity],
    ):
        result = await workflow_env.client.execute_workflow(
            ValidationWorkflow.run,
            {"valid": False},
            id="validation-wf",
            task_queue="test",
        )
        assert "validation-failed" in result
```

## 多重 Activity 工作流程測試

### 循序 Activity 模式

```python
@pytest.mark.asyncio
async def test_workflow_sequential_activities(workflow_env):
    """測試編排多個 activity 的工作流程"""

    activity_calls = []

    @activity.defn
    async def step_1(input: str) -> str:
        activity_calls.append("step_1")
        return f"{input}-step1"

    @activity.defn
    async def step_2(input: str) -> str:
        activity_calls.append("step_2")
        return f"{input}-step2"

    @activity.defn
    async def step_3(input: str) -> str:
        activity_calls.append("step_3")
        return f"{input}-step3"

    @workflow.defn
    class SequentialWorkflow:
        @workflow.run
        async def run(self, input: str) -> str:
            result_1 = await workflow.execute_activity(
                step_1,
                input,
                start_to_close_timeout=timedelta(seconds=10),
            )
            result_2 = await workflow.execute_activity(
                step_2,
                result_1,
                start_to_close_timeout=timedelta(seconds=10),
            )
            result_3 = await workflow.execute_activity(
                step_3,
                result_2,
                start_to_close_timeout=timedelta(seconds=10),
            )
            return result_3

    async with Worker(
        workflow_env.client,
        task_queue="test",
        workflows=[SequentialWorkflow],
        activities=[step_1, step_2, step_3],
    ):
        result = await workflow_env.client.execute_workflow(
            SequentialWorkflow.run,
            "start",
            id="seq-wf",
            task_queue="test",
        )
        assert result == "start-step1-step2-step3"
        assert activity_calls == ["step_1", "step_2", "step_3"]
```

### 平行 Activity 模式

```python
@pytest.mark.asyncio
async def test_workflow_parallel_activities(workflow_env):
    """測試並行 activity 執行"""

    @activity.defn
    async def parallel_task(task_id: int) -> str:
        return f"task-{task_id}"

    @workflow.defn
    class ParallelWorkflow:
        @workflow.run
        async def run(self, task_count: int) -> list[str]:
            # 並行執行 activity
            tasks = [
                workflow.execute_activity(
                    parallel_task,
                    i,
                    start_to_close_timeout=timedelta(seconds=10),
                )
                for i in range(task_count)
            ]
            return await asyncio.gather(*tasks)

    async with Worker(
        workflow_env.client,
        task_queue="test",
        workflows=[ParallelWorkflow],
        activities=[parallel_task],
    ):
        result = await workflow_env.client.execute_workflow(
            ParallelWorkflow.run,
            3,
            id="parallel-wf",
            task_queue="test",
        )
        assert result == ["task-0", "task-1", "task-2"]
```

## Signal 與 Query 測試

### Signal 處理器

```python
@pytest.mark.asyncio
async def test_workflow_signals(workflow_env):
    """測試工作流程 signal 處理"""

    @workflow.defn
    class SignalWorkflow:
        def __init__(self) -> None:
            self._status = "initialized"

        @workflow.run
        async def run(self) -> str:
            # 等待完成 signal
            await workflow.wait_condition(lambda: self._status == "completed")
            return self._status

        @workflow.signal
        async def update_status(self, new_status: str) -> None:
            self._status = new_status

        @workflow.query
        def get_status(self) -> str:
            return self._status

    async with Worker(
        workflow_env.client,
        task_queue="test",
        workflows=[SignalWorkflow],
    ):
        # 啟動工作流程
        handle = await workflow_env.client.start_workflow(
            SignalWorkflow.run,
            id="signal-wf",
            task_queue="test",
        )

        # 透過 query 驗證初始狀態
        initial_status = await handle.query(SignalWorkflow.get_status)
        assert initial_status == "initialized"

        # 發送 signal
        await handle.signal(SignalWorkflow.update_status, "processing")

        # 驗證更新後的狀態
        updated_status = await handle.query(SignalWorkflow.get_status)
        assert updated_status == "processing"

        # 完成工作流程
        await handle.signal(SignalWorkflow.update_status, "completed")
        result = await handle.result()
        assert result == "completed"
```

## 覆蓋率策略

### 工作流程邏輯覆蓋率

**目標**：≥80% 的工作流程決策邏輯覆蓋率

```python
# 測試所有分支
@pytest.mark.parametrize("condition,expected", [
    (True, "branch-a"),
    (False, "branch-b"),
])
async def test_workflow_branches(workflow_env, condition, expected):
    """確保所有程式碼路徑都被測試"""
    # 測試實作
    pass
```

### Activity 覆蓋率

**目標**：≥80% 的 activity 邏輯覆蓋率

```python
# 測試 activity 邊界案例
@pytest.mark.parametrize("input,expected", [
    ("valid", "success"),
    ("", "empty-input-error"),
    (None, "null-input-error"),
])
async def test_activity_edge_cases(activity_env, input, expected):
    """測試 activity 錯誤處理"""
    # 測試實作
    pass
```

## 整合測試組織

### 測試結構

```
tests/
├── integration/
│   ├── conftest.py              # 共用 fixture
│   ├── test_order_workflow.py   # 訂單處理測試
│   ├── test_payment_workflow.py # 付款測試
│   └── test_fulfillment_workflow.py
├── unit/
│   ├── test_order_activities.py
│   └── test_payment_activities.py
└── fixtures/
    └── test_data.py             # 測試資料建構器
```

### 共用 Fixture

```python
# conftest.py
import pytest
from temporalio.testing import WorkflowEnvironment

@pytest.fixture(scope="session")
async def workflow_env():
    """Session 範圍的整合測試環境"""
    env = await WorkflowEnvironment.start_time_skipping()
    yield env
    await env.shutdown()

@pytest.fixture
def mock_payment_service():
    """模擬外部付款服務"""
    return Mock()

@pytest.fixture
def mock_inventory_service():
    """模擬外部庫存服務"""
    return Mock()
```

## 最佳實踐

1. **模擬外部相依性**：測試時絕不呼叫真實 API
2. **測試錯誤場景**：驗證補償與重試邏輯
3. **平行測試**：使用 pytest-xdist 加快測試執行速度
4. **獨立測試**：每個測試都應該獨立
5. **清晰的斷言**：驗證結果與副作用
6. **覆蓋率目標**：關鍵工作流程 ≥80%
7. **快速執行**：使用時間跳過功能，避免真實延遲

## 額外資源

- 模擬策略：docs.temporal.io/develop/python/testing-suite
- pytest 最佳實踐：docs.pytest.org/en/stable/goodpractices.html
- Python SDK 範例：github.com/temporalio/samples-python
