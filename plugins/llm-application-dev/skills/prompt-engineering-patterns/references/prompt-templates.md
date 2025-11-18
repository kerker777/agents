# 提示詞模板系統

## 模板架構

### 基本模板結構
```python
class PromptTemplate:
    def __init__(self, template_string, variables=None):
        self.template = template_string
        self.variables = variables or []

    def render(self, **kwargs):
        missing = set(self.variables) - set(kwargs.keys())
        if missing:
            raise ValueError(f"Missing required variables: {missing}")

        return self.template.format(**kwargs)

# Usage
template = PromptTemplate(
    template_string="將 {text} 從 {source_lang} 翻譯為 {target_lang}",
    variables=['text', 'source_lang', 'target_lang']
)

prompt = template.render(
    text="Hello world",
    source_lang="English",
    target_lang="Spanish"
)
```

### 條件式模板
```python
class ConditionalTemplate(PromptTemplate):
    def render(self, **kwargs):
        # Process conditional blocks
        result = self.template

        # Handle if-blocks: {{#if variable}}content{{/if}}
        import re
        if_pattern = r'\{\{#if (\w+)\}\}(.*?)\{\{/if\}\}'

        def replace_if(match):
            var_name = match.group(1)
            content = match.group(2)
            return content if kwargs.get(var_name) else ''

        result = re.sub(if_pattern, replace_if, result, flags=re.DOTALL)

        # Handle for-loops: {{#each items}}{{this}}{{/each}}
        each_pattern = r'\{\{#each (\w+)\}\}(.*?)\{\{/each\}\}'

        def replace_each(match):
            var_name = match.group(1)
            content = match.group(2)
            items = kwargs.get(var_name, [])
            return '\n'.join(content.replace('{{this}}', str(item)) for item in items)

        result = re.sub(each_pattern, replace_each, result, flags=re.DOTALL)

        # Finally, render remaining variables
        return result.format(**kwargs)

# Usage
template = ConditionalTemplate("""
分析以下文字：
{text}

{{#if include_sentiment}}
提供情感分析。
{{/if}}

{{#if include_entities}}
擷取命名實體。
{{/if}}

{{#if examples}}
參考範例：
{{#each examples}}
- {{this}}
{{/each}}
{{/if}}
""")
```

### 模組化模板組合
```python
class ModularTemplate:
    def __init__(self):
        self.components = {}

    def register_component(self, name, template):
        self.components[name] = template

    def render(self, structure, **kwargs):
        parts = []
        for component_name in structure:
            if component_name in self.components:
                component = self.components[component_name]
                parts.append(component.format(**kwargs))

        return '\n\n'.join(parts)

# Usage
builder = ModularTemplate()

builder.register_component('system', "您是一位 {role}。")
builder.register_component('context', "上下文：{context}")
builder.register_component('instruction', "任務：{task}")
builder.register_component('examples', "範例：\n{examples}")
builder.register_component('input', "輸入：{input}")
builder.register_component('format', "輸出格式：{format}")

# Compose different templates for different scenarios
basic_prompt = builder.render(
    ['system', 'instruction', 'input'],
    role='helpful assistant',
    instruction='摘要文字',
    input='...'
)

advanced_prompt = builder.render(
    ['system', 'context', 'examples', 'instruction', 'input', 'format'],
    role='expert analyst',
    context='財務分析',
    examples='...',
    instruction='分析情感',
    input='...',
    format='JSON'
)
```

## 常見模板模式

### 分類模板
```python
CLASSIFICATION_TEMPLATE = """
將以下 {content_type} 分類到以下類別之一：{categories}

{{#if description}}
類別描述：
{description}
{{/if}}

{{#if examples}}
範例：
{examples}
{{/if}}

{content_type}：{input}

類別："""
```

### 擷取模板
```python
EXTRACTION_TEMPLATE = """
從 {content_type} 中擷取結構化資訊。

必填欄位：
{field_definitions}

{{#if examples}}
範例擷取：
{examples}
{{/if}}

{content_type}：{input}

擷取的資訊（JSON）："""
```

### 生成模板
```python
GENERATION_TEMPLATE = """
根據以下 {input_type} 生成 {output_type}。

需求：
{requirements}

{{#if style}}
風格：{style}
{{/if}}

{{#if constraints}}
限制：
{constraints}
{{/if}}

{{#if examples}}
範例：
{examples}
{{/if}}

{input_type}：{input}

{output_type}："""
```

### 轉換模板
```python
TRANSFORMATION_TEMPLATE = """
將輸入的 {source_format} 轉換為 {target_format}。

轉換規則：
{rules}

{{#if examples}}
範例轉換：
{examples}
{{/if}}

輸入 {source_format}：
{input}

輸出 {target_format}："""
```

## 進階功能

### 模板繼承
```python
class TemplateRegistry:
    def __init__(self):
        self.templates = {}

    def register(self, name, template, parent=None):
        if parent and parent in self.templates:
            # Inherit from parent
            base = self.templates[parent]
            template = self.merge_templates(base, template)

        self.templates[name] = template

    def merge_templates(self, parent, child):
        # Child overwrites parent sections
        return {**parent, **child}

# Usage
registry = TemplateRegistry()

registry.register('base_analysis', {
    'system': '您是一位專業分析師。',
    'format': '以結構化格式提供分析。'
})

registry.register('sentiment_analysis', {
    'instruction': '分析情感',
    'format': '提供從 -1 到 1 的情感分數。'
}, parent='base_analysis')
```

### 變數驗證
```python
class ValidatedTemplate:
    def __init__(self, template, schema):
        self.template = template
        self.schema = schema

    def validate_vars(self, **kwargs):
        for var_name, var_schema in self.schema.items():
            if var_name in kwargs:
                value = kwargs[var_name]

                # Type validation
                if 'type' in var_schema:
                    expected_type = var_schema['type']
                    if not isinstance(value, expected_type):
                        raise TypeError(f"{var_name} must be {expected_type}")

                # Range validation
                if 'min' in var_schema and value < var_schema['min']:
                    raise ValueError(f"{var_name} must be >= {var_schema['min']}")

                if 'max' in var_schema and value > var_schema['max']:
                    raise ValueError(f"{var_name} must be <= {var_schema['max']}")

                # Enum validation
                if 'choices' in var_schema and value not in var_schema['choices']:
                    raise ValueError(f"{var_name} must be one of {var_schema['choices']}")

    def render(self, **kwargs):
        self.validate_vars(**kwargs)
        return self.template.format(**kwargs)

# Usage
template = ValidatedTemplate(
    template="以 {length} 字摘要，使用 {tone} 語氣",
    schema={
        'length': {'type': int, 'min': 10, 'max': 500},
        'tone': {'type': str, 'choices': ['formal', 'casual', 'technical']}
    }
)
```

### 模板快取
```python
class CachedTemplate:
    def __init__(self, template):
        self.template = template
        self.cache = {}

    def render(self, use_cache=True, **kwargs):
        if use_cache:
            cache_key = self.get_cache_key(kwargs)
            if cache_key in self.cache:
                return self.cache[cache_key]

        result = self.template.format(**kwargs)

        if use_cache:
            self.cache[cache_key] = result

        return result

    def get_cache_key(self, kwargs):
        return hash(frozenset(kwargs.items()))

    def clear_cache(self):
        self.cache = {}
```

## 多輪模板

### 對話模板
```python
class ConversationTemplate:
    def __init__(self, system_prompt):
        self.system_prompt = system_prompt
        self.history = []

    def add_user_message(self, message):
        self.history.append({'role': 'user', 'content': message})

    def add_assistant_message(self, message):
        self.history.append({'role': 'assistant', 'content': message})

    def render_for_api(self):
        messages = [{'role': 'system', 'content': self.system_prompt}]
        messages.extend(self.history)
        return messages

    def render_as_text(self):
        result = f"系統：{self.system_prompt}\n\n"
        for msg in self.history:
            role = msg['role'].capitalize()
            result += f"{role}：{msg['content']}\n\n"
        return result
```

### 基於狀態的模板
```python
class StatefulTemplate:
    def __init__(self):
        self.state = {}
        self.templates = {}

    def set_state(self, **kwargs):
        self.state.update(kwargs)

    def register_state_template(self, state_name, template):
        self.templates[state_name] = template

    def render(self):
        current_state = self.state.get('current_state', 'default')
        template = self.templates.get(current_state)

        if not template:
            raise ValueError(f"No template for state: {current_state}")

        return template.format(**self.state)

# Usage for multi-step workflows
workflow = StatefulTemplate()

workflow.register_state_template('init', """
歡迎！讓我們 {task}。
您的 {first_input} 是什麼？
""")

workflow.register_state_template('processing', """
謝謝！正在處理 {first_input}。
現在，您的 {second_input} 是什麼？
""")

workflow.register_state_template('complete', """
太好了！根據：
- {first_input}
- {second_input}

這是結果：{result}
""")
```

## 最佳實務

1. **保持 DRY**：使用模板避免重複
2. **提早驗證**：在渲染前檢查變數
3. **版本化模板**：像程式碼一樣追蹤變更
4. **測試變化**：確保模板適用於多樣化的輸入
5. **記錄變數**：清楚指定必填/選填變數
6. **使用型別提示**：明確指定變數型別
7. **提供預設值**：在適當的地方設定合理的預設值
8. **明智地快取**：快取靜態模板，而非動態模板

## 模板庫

### 問答
```python
QA_TEMPLATES = {
    'factual': """根據上下文回答問題。

上下文：{context}
問題：{question}
答案：""",

    'multi_hop': """透過跨多個事實推理來回答問題。

事實：{facts}
問題：{question}

推理：""",

    'conversational': """自然地延續對話。

先前的對話：
{history}

使用者：{question}
助理："""
}
```

### 內容生成
```python
GENERATION_TEMPLATES = {
    'blog_post': """撰寫一篇關於 {topic} 的部落格文章。

需求：
- 長度：{word_count} 字
- 語氣：{tone}
- 包含：{key_points}

部落格文章：""",

    'product_description': """為 {product} 撰寫產品描述。

特色：{features}
優點：{benefits}
目標受眾：{audience}

描述：""",

    'email': """撰寫一封 {type} 電子郵件。

收件人：{recipient}
情境：{context}
重點：{key_points}

電子郵件："""
}
```

## 效能考量

- 為重複使用預先編譯模板
- 當變數是靜態的時快取已渲染的模板
- 最小化迴圈中的字串串接
- 使用高效的字串格式化（f-strings、.format()）
- 分析模板渲染的瓶頸
