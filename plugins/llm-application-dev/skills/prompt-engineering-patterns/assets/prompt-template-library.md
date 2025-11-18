# 提示詞模板庫

## 分類模板

### 情感分析
```
將以下文字的情感分類為正面、負面或中性。

文字：{text}

情感：
```

### 意圖偵測
```
從以下訊息中判斷使用者的意圖。

可能的意圖：{intent_list}

訊息：{message}

意圖：
```

### 主題分類
```
將以下文章分類到以下類別之一：{categories}

文章：
{article}

類別：
```

## 擷取模板

### 命名實體識別
```
從文字中擷取所有命名實體並進行分類。

文字：{text}

實體（JSON 格式）：
{
  "persons": [],
  "organizations": [],
  "locations": [],
  "dates": []
}
```

### 結構化資料擷取
```
從職位公告中擷取結構化資訊。

職位公告：
{posting}

擷取的資訊（JSON）：
{
  "title": "",
  "company": "",
  "location": "",
  "salary_range": "",
  "requirements": [],
  "responsibilities": []
}
```

## 生成模板

### 電子郵件生成
```
撰寫一封專業的{email_type}電子郵件。

收件人：{recipient}
情境：{context}
要包含的重點：
{key_points}

電子郵件：
主旨：
內容：
```

### 程式碼生成
```
為以下任務生成{language}程式碼：

任務：{task_description}

需求：
{requirements}

包含：
- 錯誤處理
- 輸入驗證
- 行內註解

程式碼：
```

### 創意寫作
```
撰寫一個{length}字的{style}故事，主題是{topic}。

包含以下元素：
- {element_1}
- {element_2}
- {element_3}

故事：
```

## 轉換模板

### 摘要
```
將以下文字摘要為{num_sentences}句話。

文字：
{text}

摘要：
```

### 含上下文的翻譯
```
將以下{source_lang}文字翻譯為{target_lang}。

情境：{context}
語氣：{tone}

文字：{text}

翻譯：
```

### 格式轉換
```
將以下{source_format}轉換為{target_format}。

輸入：
{input_data}

輸出（{target_format}）：
```

## 分析模板

### 程式碼審查
```
審查以下程式碼，檢查：
1. 錯誤和問題
2. 效能問題
3. 安全漏洞
4. 最佳實務違規

程式碼：
{code}

審查：
```

### SWOT 分析
```
針對以下主題進行 SWOT 分析：{subject}

情境：{context}

分析：
優勢：
-

劣勢：
-

機會：
-

威脅：
-
```

## 問答模板

### RAG 模板
```
根據提供的上下文回答問題。如果上下文不包含足夠資訊，請明確說明。

上下文：
{context}

問題：{question}

答案：
```

### 多輪問答
```
先前的對話：
{conversation_history}

新問題：{question}

答案（自然地從對話延續）：
```

## 專門模板

### SQL 查詢生成
```
為以下請求生成 SQL 查詢。

資料庫架構：
{schema}

請求：{request}

SQL 查詢：
```

### 正規表示式模式建立
```
建立一個正規表示式模式來匹配：{requirement}

應該匹配的測試案例：
{positive_examples}

不應該匹配的測試案例：
{negative_examples}

正規表示式模式：
```

### API 文件
```
為此函式生成 API 文件：

程式碼：
{function_code}

文件（遵循{doc_format}格式）：
```

## 透過填入 {變數} 來使用這些模板
