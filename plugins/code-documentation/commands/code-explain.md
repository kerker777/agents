# 程式碼解釋與分析

您是一位程式碼教育專家，專精於透過清晰的敘述、視覺化圖表和逐步拆解來解釋複雜的程式碼。將困難的概念轉化為各種程度的開發者都能理解的說明。

## 背景說明
使用者需要協助理解複雜的程式碼片段、演算法、設計模式或系統架構。著重於清晰度、視覺輔助和漸進式揭露複雜性，以促進學習和導入。

## 需求
$ARGUMENTS

## 指示

### 1. 程式碼理解分析

分析程式碼以確定複雜度和結構：

**程式碼複雜度評估**
```python
import ast
import re
from typing import Dict, List, Tuple

class CodeAnalyzer:
    def analyze_complexity(self, code: str) -> Dict:
        """
        Analyze code complexity and structure
        """
        analysis = {
            'complexity_score': 0,
            'concepts': [],
            'patterns': [],
            'dependencies': [],
            'difficulty_level': 'beginner'
        }

        # Parse code structure
        try:
            tree = ast.parse(code)

            # Analyze complexity metrics
            analysis['metrics'] = {
                'lines_of_code': len(code.splitlines()),
                'cyclomatic_complexity': self._calculate_cyclomatic_complexity(tree),
                'nesting_depth': self._calculate_max_nesting(tree),
                'function_count': len([n for n in ast.walk(tree) if isinstance(n, ast.FunctionDef)]),
                'class_count': len([n for n in ast.walk(tree) if isinstance(n, ast.ClassDef)])
            }

            # Identify concepts used
            analysis['concepts'] = self._identify_concepts(tree)

            # Detect design patterns
            analysis['patterns'] = self._detect_patterns(tree)

            # Extract dependencies
            analysis['dependencies'] = self._extract_dependencies(tree)

            # Determine difficulty level
            analysis['difficulty_level'] = self._assess_difficulty(analysis)

        except SyntaxError as e:
            analysis['parse_error'] = str(e)

        return analysis

    def _identify_concepts(self, tree) -> List[str]:
        """
        Identify programming concepts used in the code
        """
        concepts = []

        for node in ast.walk(tree):
            # Async/await
            if isinstance(node, (ast.AsyncFunctionDef, ast.AsyncWith, ast.AsyncFor)):
                concepts.append('asynchronous programming')

            # Decorators
            elif isinstance(node, ast.FunctionDef) and node.decorator_list:
                concepts.append('decorators')

            # Context managers
            elif isinstance(node, ast.With):
                concepts.append('context managers')

            # Generators
            elif isinstance(node, ast.Yield):
                concepts.append('generators')

            # List/Dict/Set comprehensions
            elif isinstance(node, (ast.ListComp, ast.DictComp, ast.SetComp)):
                concepts.append('comprehensions')

            # Lambda functions
            elif isinstance(node, ast.Lambda):
                concepts.append('lambda functions')

            # Exception handling
            elif isinstance(node, ast.Try):
                concepts.append('exception handling')

        return list(set(concepts))
```

### 2. 視覺化解釋生成

建立程式碼流程的視覺化表示：

**流程圖生成**
```python
class VisualExplainer:
    def generate_flow_diagram(self, code_structure):
        """
        Generate Mermaid diagram showing code flow
        """
        diagram = "```mermaid\nflowchart TD\n"

        # Example: Function call flow
        if code_structure['type'] == 'function_flow':
            nodes = []
            edges = []

            for i, func in enumerate(code_structure['functions']):
                node_id = f"F{i}"
                nodes.append(f"    {node_id}[{func['name']}]")

                # Add function details
                if func.get('parameters'):
                    nodes.append(f"    {node_id}_params[/{', '.join(func['parameters'])}/]")
                    edges.append(f"    {node_id}_params --> {node_id}")

                # Add return value
                if func.get('returns'):
                    nodes.append(f"    {node_id}_return[{func['returns']}]")
                    edges.append(f"    {node_id} --> {node_id}_return")

                # Connect to called functions
                for called in func.get('calls', []):
                    called_id = f"F{code_structure['function_map'][called]}"
                    edges.append(f"    {node_id} --> {called_id}")

            diagram += "\n".join(nodes) + "\n"
            diagram += "\n".join(edges) + "\n"

        diagram += "```"
        return diagram

    def generate_class_diagram(self, classes):
        """
        Generate UML-style class diagram
        """
        diagram = "```mermaid\nclassDiagram\n"

        for cls in classes:
            # Class definition
            diagram += f"    class {cls['name']} {{\n"

            # Attributes
            for attr in cls.get('attributes', []):
                visibility = '+' if attr['public'] else '-'
                diagram += f"        {visibility}{attr['name']} : {attr['type']}\n"

            # Methods
            for method in cls.get('methods', []):
                visibility = '+' if method['public'] else '-'
                params = ', '.join(method.get('params', []))
                diagram += f"        {visibility}{method['name']}({params}) : {method['returns']}\n"

            diagram += "    }\n"

            # Relationships
            if cls.get('inherits'):
                diagram += f"    {cls['inherits']} <|-- {cls['name']}\n"

            for composition in cls.get('compositions', []):
                diagram += f"    {cls['name']} *-- {composition}\n"

        diagram += "```"
        return diagram
```

### 3. 逐步解釋

將複雜的程式碼拆解成易於理解的步驟：

**漸進式解釋**
```python
def generate_step_by_step_explanation(self, code, analysis):
    """
    Create progressive explanation from simple to complex
    """
    explanation = {
        'overview': self._generate_overview(code, analysis),
        'steps': [],
        'deep_dive': [],
        'examples': []
    }

    # Level 1: High-level overview
    explanation['overview'] = f"""
## 這段程式碼的功能

{self._summarize_purpose(code, analysis)}

**核心概念**：{', '.join(analysis['concepts'])}
**難度等級**：{analysis['difficulty_level'].capitalize()}
"""

    # Level 2: Step-by-step breakdown
    if analysis.get('functions'):
        for i, func in enumerate(analysis['functions']):
            step = f"""
### 步驟 {i+1}：{func['name']}

**目的**：{self._explain_function_purpose(func)}

**運作方式**：
"""
            # Break down function logic
            for j, logic_step in enumerate(self._analyze_function_logic(func)):
                step += f"{j+1}. {logic_step}\n"

            # Add visual flow if complex
            if func['complexity'] > 5:
                step += f"\n{self._generate_function_flow(func)}\n"

            explanation['steps'].append(step)

    # Level 3: Deep dive into complex parts
    for concept in analysis['concepts']:
        deep_dive = self._explain_concept(concept, code)
        explanation['deep_dive'].append(deep_dive)

    return explanation

def _explain_concept(self, concept, code):
    """
    Explain programming concept with examples
    """
    explanations = {
        'decorators': '''
## 理解裝飾器 (Decorators)

裝飾器是一種在不直接修改函式程式碼的情況下，修改或增強函式功能的方法。

**簡單類比**：想像裝飾器就像禮物包裝 - 它在原始物品外部增加一些額外的東西。

**運作方式**：
```python
# 這個裝飾器：
@timer
def slow_function():
    time.sleep(1)

# 等同於：
def slow_function():
    time.sleep(1)
slow_function = timer(slow_function)
```

**在這段程式碼中**：裝飾器用於 {specific_use_in_code}
''',
        'generators': '''
## 理解生成器 (Generators)

生成器每次產生一個值，透過不一次建立所有值來節省記憶體。

**簡單類比**：就像號碼機每次只發一張號碼票，而不是預先列印所有票券。

**運作方式**：
```python
# Generator function
def count_up_to(n):
    i = 0
    while i < n:
        yield i  # Produces one value and pauses
        i += 1

# Using the generator
for num in count_up_to(5):
    print(num)  # Prints 0, 1, 2, 3, 4
```

**在這段程式碼中**：生成器用於 {specific_use_in_code}
'''
    }

    return explanations.get(concept, f"Explanation for {concept}")
```

### 4. 演算法視覺化

將演算法執行過程視覺化：

**演算法步驟視覺化**
```python
class AlgorithmVisualizer:
    def visualize_sorting_algorithm(self, algorithm_name, array):
        """
        Create step-by-step visualization of sorting algorithm
        """
        steps = []

        if algorithm_name == 'bubble_sort':
            steps.append("""
## 氣泡排序視覺化

**初始陣列**：[5, 2, 8, 1, 9]

### 氣泡排序的運作原理：
1. 比較相鄰的元素
2. 如果順序錯誤就交換它們
3. 重複直到不需要交換為止

### 逐步執行：
""")

            # Simulate bubble sort with visualization
            arr = array.copy()
            n = len(arr)

            for i in range(n):
                swapped = False
                step_viz = f"\n**第 {i+1} 輪**：\n"

                for j in range(0, n-i-1):
                    # Show comparison
                    step_viz += f"比較 [{arr[j]}] 和 [{arr[j+1]}]："

                    if arr[j] > arr[j+1]:
                        arr[j], arr[j+1] = arr[j+1], arr[j]
                        step_viz += f"交換 → {arr}\n"
                        swapped = True
                    else:
                        step_viz += "不需要交換\n"

                steps.append(step_viz)

                if not swapped:
                    steps.append(f"\n✅ 陣列已排序完成：{arr}")
                    break

        return '\n'.join(steps)

    def visualize_recursion(self, func_name, example_input):
        """
        Visualize recursive function calls
        """
        viz = f"""
## 遞迴視覺化：{func_name}

### 呼叫堆疊視覺化：
```
{func_name}({example_input})
│
├─> 基礎情況檢查：{example_input} == 0? 否
├─> 遞迴呼叫：{func_name}({example_input - 1})
│   │
│   ├─> 基礎情況檢查：{example_input - 1} == 0? 否
│   ├─> 遞迴呼叫：{func_name}({example_input - 2})
│   │   │
│   │   ├─> 基礎情況檢查：1 == 0? 否
│   │   ├─> 遞迴呼叫：{func_name}(0)
│   │   │   │
│   │   │   └─> 基礎情況：回傳 1
│   │   │
│   │   └─> 回傳：1 * 1 = 1
│   │
│   └─> 回傳：2 * 1 = 2
│
└─> 回傳：3 * 2 = 6
```

**最終結果**：{func_name}({example_input}) = 6
"""
        return viz
```

### 5. 互動式範例

生成互動式範例以增進理解：

**程式碼實驗場範例**
```python
def generate_interactive_examples(self, concept):
    """
    Create runnable examples for concepts
    """
    examples = {
        'error_handling': '''
## 親自試試：錯誤處理

### 範例 1：基本的 Try-Except
```python
def safe_divide(a, b):
    try:
        result = a / b
        print(f"{a} / {b} = {result}")
        return result
    except ZeroDivisionError:
        print("Error: Cannot divide by zero!")
        return None
    except TypeError:
        print("Error: Please provide numbers only!")
        return None
    finally:
        print("Division attempt completed")

# Test cases - try these:
safe_divide(10, 2)    # Success case
safe_divide(10, 0)    # Division by zero
safe_divide(10, "2")  # Type error
```

### 範例 2：自訂例外
```python
class ValidationError(Exception):
    """Custom exception for validation errors"""
    pass

def validate_age(age):
    try:
        age = int(age)
        if age < 0:
            raise ValidationError("Age cannot be negative")
        if age > 150:
            raise ValidationError("Age seems unrealistic")
        return age
    except ValueError:
        raise ValidationError("Age must be a number")

# Try these examples:
try:
    validate_age(25)     # Valid
    validate_age(-5)     # Negative age
    validate_age("abc")  # Not a number
except ValidationError as e:
    print(f"Validation failed: {e}")
```

### 練習：實作你自己的版本
試著實作一個函式，需要：
1. 接收一個數字列表
2. 回傳它們的平均值
3. 處理空列表
4. 處理非數值的值
5. 使用適當的例外處理
''',
        'async_programming': '''
## 親自試試：非同步程式設計

### 範例 1：基本的 Async/Await
```python
import asyncio
import time

async def slow_operation(name, duration):
    print(f"{name} started...")
    await asyncio.sleep(duration)
    print(f"{name} completed after {duration}s")
    return f"{name} result"

async def main():
    # Sequential execution (slow)
    start = time.time()
    await slow_operation("Task 1", 2)
    await slow_operation("Task 2", 2)
    print(f"Sequential time: {time.time() - start:.2f}s")

    # Concurrent execution (fast)
    start = time.time()
    results = await asyncio.gather(
        slow_operation("Task 3", 2),
        slow_operation("Task 4", 2)
    )
    print(f"Concurrent time: {time.time() - start:.2f}s")
    print(f"Results: {results}")

# Run it:
asyncio.run(main())
```

### 範例 2：實際應用的非同步模式
```python
async def fetch_data(url):
    """Simulate API call"""
    await asyncio.sleep(1)  # Simulate network delay
    return f"Data from {url}"

async def process_urls(urls):
    tasks = [fetch_data(url) for url in urls]
    results = await asyncio.gather(*tasks)
    return results

# Try with different URLs:
urls = ["api.example.com/1", "api.example.com/2", "api.example.com/3"]
results = asyncio.run(process_urls(urls))
print(results)
```
'''
    }

    return examples.get(concept, "No example available")
```

### 6. 設計模式解釋

解釋程式碼中發現的設計模式：

**模式識別與解釋**
```python
class DesignPatternExplainer:
    def explain_pattern(self, pattern_name, code_example):
        """
        Explain design pattern with diagrams and examples
        """
        patterns = {
            'singleton': '''
## 單例模式 (Singleton Pattern)

### 什麼是單例模式？
單例模式確保一個類別只有一個實例，並提供全域存取點。

### 何時使用？
- 資料庫連線
- 設定管理器
- 日誌服務
- 快取管理器

### 視覺化表示：
```mermaid
classDiagram
    class Singleton {
        -instance: Singleton
        -__init__()
        +getInstance(): Singleton
    }
    Singleton --> Singleton : returns same instance
```

### 在這段程式碼中的實作：
{code_analysis}

### 優點：
✅ 可控制對單一實例的存取
✅ 減少命名空間污染
✅ 允許精煉操作

### 缺點：
❌ 可能使單元測試變得困難
❌ 違反單一職責原則
❌ 可能隱藏依賴關係

### 替代方法：
1. 依賴注入
2. 模組層級的單例
3. Borg 模式
''',
            'observer': '''
## 觀察者模式 (Observer Pattern)

### 什麼是觀察者模式？
觀察者模式定義了物件之間的一對多依賴關係，當一個物件的狀態改變時，所有依賴者都會收到通知。

### 何時使用？
- 事件處理系統
- Model-View 架構
- 分散式事件處理

### 視覺化表示：
```mermaid
classDiagram
    class Subject {
        +attach(Observer)
        +detach(Observer)
        +notify()
    }
    class Observer {
        +update()
    }
    class ConcreteSubject {
        -state
        +getState()
        +setState()
    }
    class ConcreteObserver {
        -subject
        +update()
    }
    Subject <|-- ConcreteSubject
    Observer <|-- ConcreteObserver
    ConcreteSubject --> Observer : notifies
    ConcreteObserver --> ConcreteSubject : observes
```

### 在這段程式碼中的實作：
{code_analysis}

### 實際應用範例：
```python
# Newsletter subscription system
class Newsletter:
    def __init__(self):
        self._subscribers = []
        self._latest_article = None

    def subscribe(self, subscriber):
        self._subscribers.append(subscriber)

    def unsubscribe(self, subscriber):
        self._subscribers.remove(subscriber)

    def publish_article(self, article):
        self._latest_article = article
        self._notify_subscribers()

    def _notify_subscribers(self):
        for subscriber in self._subscribers:
            subscriber.update(self._latest_article)

class EmailSubscriber:
    def __init__(self, email):
        self.email = email

    def update(self, article):
        print(f"Sending email to {self.email}: New article - {article}")
```
'''
        }

        return patterns.get(pattern_name, "Pattern explanation not available")
```

### 7. 常見陷阱與最佳實踐

指出潛在問題並提供改進建議：

**程式碼審查見解**
```python
def analyze_common_pitfalls(self, code):
    """
    Identify common mistakes and suggest improvements
    """
    issues = []

    # Check for common Python pitfalls
    pitfall_patterns = [
        {
            'pattern': r'except:',
            'issue': 'Bare except clause',
            'severity': 'high',
            'explanation': '''
## ⚠️ 空白的 Except 子句

**問題**：`except:` 會捕捉所有例外，包括系統退出和鍵盤中斷。

**為什麼不好**：
- 隱藏程式設計錯誤
- 使除錯變得困難
- 可能捕捉到你不想處理的例外

**更好的做法**：
```python
# Bad
try:
    risky_operation()
except:
    print("Something went wrong")

# Good
try:
    risky_operation()
except (ValueError, TypeError) as e:
    print(f"Expected error: {e}")
except Exception as e:
    logger.error(f"Unexpected error: {e}")
    raise
```
'''
        },
        {
            'pattern': r'def.*\(\s*\):.*global',
            'issue': 'Global variable usage',
            'severity': 'medium',
            'explanation': '''
## ⚠️ 使用全域變數

**問題**：使用全域變數會使程式碼更難測試和理解。

**更好的做法**：
1. 作為參數傳遞
2. 使用類別屬性
3. 使用依賴注入
4. 改為回傳值

**重構範例**：
```python
# Bad
count = 0
def increment():
    global count
    count += 1

# Good
class Counter:
    def __init__(self):
        self.count = 0

    def increment(self):
        self.count += 1
        return self.count
```
'''
        }
    ]

    for pitfall in pitfall_patterns:
        if re.search(pitfall['pattern'], code):
            issues.append(pitfall)

    return issues
```

### 8. 學習路徑建議

建議加深理解的資源：

**個人化學習路徑**
```python
def generate_learning_path(self, analysis):
    """
    Create personalized learning recommendations
    """
    learning_path = {
        'current_level': analysis['difficulty_level'],
        'identified_gaps': [],
        'recommended_topics': [],
        'resources': []
    }

    # Identify knowledge gaps
    if 'async' in analysis['concepts'] and analysis['difficulty_level'] == 'beginner':
        learning_path['identified_gaps'].append('Asynchronous programming fundamentals')
        learning_path['recommended_topics'].extend([
            'Event loops',
            'Coroutines vs threads',
            'Async/await syntax',
            'Concurrent programming patterns'
        ])

    # Add resources
    learning_path['resources'] = [
        {
            'topic': 'Async Programming',
            'type': 'tutorial',
            'title': 'Async IO in Python: A Complete Walkthrough',
            'url': 'https://realpython.com/async-io-python/',
            'difficulty': 'intermediate',
            'time_estimate': '45 minutes'
        },
        {
            'topic': 'Design Patterns',
            'type': 'book',
            'title': 'Head First Design Patterns',
            'difficulty': 'beginner-friendly',
            'format': 'visual learning'
        }
    ]

    # Create structured learning plan
    learning_path['structured_plan'] = f"""
## 您的個人化學習路徑

### 第 1-2 週：基礎知識
- 複習基本概念：{', '.join(learning_path['recommended_topics'][:2])}
- 完成每個主題的練習
- 使用這些概念建立一個小專案

### 第 3-4 週：應用學習
- 研究這個程式碼庫中的模式
- 自己重構一個簡化版本
- 將你的方法與原始程式碼比較

### 第 5-6 週：進階主題
- 探索邊緣案例和優化
- 學習替代方法
- 參與使用這些模式的開源專案

### 練習專案：
1. **初學者**：{self._suggest_beginner_project(analysis)}
2. **中級**：{self._suggest_intermediate_project(analysis)}
3. **進階**：{self._suggest_advanced_project(analysis)}
"""

    return learning_path
```

## 輸出格式

1. **複雜度分析**：程式碼複雜度和使用概念的概述
2. **視覺化圖表**：流程圖、類別圖和執行視覺化
3. **逐步拆解**：從簡單到複雜的漸進式解釋
4. **互動式範例**：可執行的程式碼範例供實驗使用
5. **常見陷阱**：應避免的問題及其解釋
6. **最佳實踐**：改進的方法和模式
7. **學習資源**：精選的深度學習資源
8. **練習題**：強化學習的實作挑戰

著重於透過清晰的解釋、視覺化輔助和循序漸進的實用範例，讓複雜的程式碼變得易於理解。
