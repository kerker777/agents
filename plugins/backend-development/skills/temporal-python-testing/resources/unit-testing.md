# Temporal Workflows 和 Activities 的單元測試

針對使用 WorkflowEnvironment 和 ActivityEnvironment 獨立測試個別 workflows 和 activities 的重點指南。

## WorkflowEnvironment 與時間跳躍

**用途**：以即時的時間推進方式獨立測試 workflows（月級 workflows → 秒級）

### 基本設定模式

```python
import pytest
from temporalio.testing import WorkflowEnvironment
from temporalio.worker import Worker

@pytest.fixture
async def workflow_env():
    """可重複使用的時間跳躍測試環境"""
    env = await WorkflowEnvironment.start_time_skipping()
    yield env
    await env.shutdown()

@pytest.mark.asyncio
async def test_workflow_execution(workflow_env):
    """使用時間跳躍測試 workflow"""
    async with Worker(
        workflow_env.client,
        task_queue="test-queue",
        workflows=[YourWorkflow],
        activities=[your_activity],
    ):
        result = await workflow_env.client.execute_workflow(
            YourWorkflow.run,
            "test-input",
            id="test-wf-id",
            task_queue="test-queue",
        )
        assert result == "expected-output"
```

**主要優勢**：
- `workflow.sleep(timedelta(days=30))` 立即完成
- 快速反饋循環（毫秒 vs 小時）
- 確定性測試執行

### 時間跳躍範例

**Sleep 推進**：
```python
@pytest.mark.asyncio
async def test_workflow_with_delays(workflow_env):
    """在時間跳躍模式下，workflow sleeps 是即時的"""

    @workflow.defn
    class DelayedWorkflow:
        @workflow.run
        async def run(self) -> str:
            await workflow.sleep(timedelta(hours=24))  # 在測試中即時完成
            return "completed"

    async with Worker(
        workflow_env.client,
        task_queue="test",
        workflows=[DelayedWorkflow],
    ):
        result = await workflow_env.client.execute_workflow(
            DelayedWorkflow.run,
            id="delayed-wf",
            task_queue="test",
        )
        assert result == "completed"
```

**手動時間控制**：
```python
@pytest.mark.asyncio
async def test_workflow_manual_time(workflow_env):
    """手動推進時間以進行精確控制"""

    handle = await workflow_env.client.start_workflow(
        TimeBasedWorkflow.run,
        id="time-wf",
        task_queue="test",
    )

    # 按特定時間量推進時間
    await workflow_env.sleep(timedelta(hours=1))

    # 透過查詢驗證中間狀態
    state = await handle.query(TimeBasedWorkflow.get_state)
    assert state == "processing"

    # 推進至完成
    await workflow_env.sleep(timedelta(hours=23))
    result = await handle.result()
    assert result == "completed"
```

### 測試 Workflow 邏輯

**決策測試**：
```python
@pytest.mark.asyncio
async def test_workflow_branching(workflow_env):
    """測試不同的執行路徑"""

    @workflow.defn
    class ConditionalWorkflow:
        @workflow.run
        async def run(self, condition: bool) -> str:
            if condition:
                return "path-a"
            return "path-b"

    async with Worker(
        workflow_env.client,
        task_queue="test",
        workflows=[ConditionalWorkflow],
    ):
        # 測試 true 路徑
        result_a = await workflow_env.client.execute_workflow(
            ConditionalWorkflow.run,
            True,
            id="cond-wf-true",
            task_queue="test",
        )
        assert result_a == "path-a"

        # 測試 false 路徑
        result_b = await workflow_env.client.execute_workflow(
            ConditionalWorkflow.run,
            False,
            id="cond-wf-false",
            task_queue="test",
        )
        assert result_b == "path-b"
```

## ActivityEnvironment 測試

**用途**：在沒有 workflows 或 Temporal server 的情況下獨立測試 activities

### 基本 Activity 測試

```python
from temporalio.testing import ActivityEnvironment

async def test_activity_basic():
    """在沒有 workflow 上下文的情況下測試 activity"""

    @activity.defn
    async def process_data(input: str) -> str:
        return input.upper()

    env = ActivityEnvironment()
    result = await env.run(process_data, "test")
    assert result == "TEST"
```

### 測試 Activity 上下文

**Heartbeat 測試**：
```python
async def test_activity_heartbeat():
    """驗證 heartbeat 呼叫"""

    @activity.defn
    async def long_running_activity(total_items: int) -> int:
        for i in range(total_items):
            activity.heartbeat(i)  # 回報進度
            await asyncio.sleep(0.1)
        return total_items

    env = ActivityEnvironment()
    result = await env.run(long_running_activity, 10)
    assert result == 10
```

**取消測試**：
```python
async def test_activity_cancellation():
    """測試 activity 取消處理"""

    @activity.defn
    async def cancellable_activity() -> str:
        try:
            while True:
                if activity.is_cancelled():
                    return "cancelled"
                await asyncio.sleep(0.1)
        except asyncio.CancelledError:
            return "cancelled"

    env = ActivityEnvironment(cancellation_reason="test-cancel")
    result = await env.run(cancellable_activity)
    assert result == "cancelled"
```

### 測試錯誤處理

**例外傳播**：
```python
async def test_activity_error():
    """測試 activity 錯誤處理"""

    @activity.defn
    async def failing_activity(should_fail: bool) -> str:
        if should_fail:
            raise ApplicationError("Validation failed", non_retryable=True)
        return "success"

    env = ActivityEnvironment()

    # 測試成功路徑
    result = await env.run(failing_activity, False)
    assert result == "success"

    # 測試錯誤路徑
    with pytest.raises(ApplicationError) as exc_info:
        await env.run(failing_activity, True)
    assert "Validation failed" in str(exc_info.value)
```

## Pytest 整合模式

### 共用 Fixtures

```python
# conftest.py
import pytest
from temporalio.testing import WorkflowEnvironment

@pytest.fixture(scope="module")
async def workflow_env():
    """模組範圍的環境（在測試間重複使用）"""
    env = await WorkflowEnvironment.start_time_skipping()
    yield env
    await env.shutdown()

@pytest.fixture
def activity_env():
    """函式範圍的環境（每個測試都是新的）"""
    return ActivityEnvironment()
```

### 參數化測試

```python
@pytest.mark.parametrize("input,expected", [
    ("test", "TEST"),
    ("hello", "HELLO"),
    ("123", "123"),
])
async def test_activity_parameterized(activity_env, input, expected):
    """測試多個輸入情境"""
    result = await activity_env.run(process_data, input)
    assert result == expected
```

## 最佳實踐

1. **快速執行**：對所有 workflow 測試使用時間跳躍
2. **隔離**：分別測試 workflows 和 activities
3. **共用 Fixtures**：在相關測試間重複使用 WorkflowEnvironment
4. **覆蓋率目標**：workflow 邏輯 ≥80%
5. **模擬 Activities**：對 activity 特定邏輯使用 ActivityEnvironment
6. **確定性**：確保測試結果在多次執行間保持一致
7. **錯誤案例**：測試成功和失敗兩種情境

## 常見模式

**測試重試邏輯**：
```python
@pytest.mark.asyncio
async def test_workflow_with_retries(workflow_env):
    """測試 activity 重試行為"""

    call_count = 0

    @activity.defn
    async def flaky_activity() -> str:
        nonlocal call_count
        call_count += 1
        if call_count < 3:
            raise Exception("Transient error")
        return "success"

    @workflow.defn
    class RetryWorkflow:
        @workflow.run
        async def run(self) -> str:
            return await workflow.execute_activity(
                flaky_activity,
                start_to_close_timeout=timedelta(seconds=10),
                retry_policy=RetryPolicy(
                    initial_interval=timedelta(milliseconds=1),
                    maximum_attempts=5,
                ),
            )

    async with Worker(
        workflow_env.client,
        task_queue="test",
        workflows=[RetryWorkflow],
        activities=[flaky_activity],
    ):
        result = await workflow_env.client.execute_workflow(
            RetryWorkflow.run,
            id="retry-wf",
            task_queue="test",
        )
        assert result == "success"
        assert call_count == 3  # 驗證重試次數

```

## 其他資源

- Python SDK Testing: docs.temporal.io/develop/python/testing-suite
- pytest Documentation: docs.pytest.org
- Temporal Samples: github.com/temporalio/samples-python
