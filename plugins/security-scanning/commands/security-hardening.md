透過協同式多代理編排實現全面的安全加固與縱深防禦策略：

[延伸思考：此工作流程在所有應用程式層級實施縱深防禦安全策略。它協調專業的安全代理來執行全面評估、實施分層安全控制，並建立持續性的安全監控。此方法遵循現代 DevSecOps 原則，包括左移安全、自動化掃描和合規性驗證。每個階段都建立在先前發現的基礎上，創建能夠應對當前漏洞和未來威脅的彈性安全態勢。]

## 階段 1：全面安全評估

### 1. 初始漏洞掃描
- 使用 Task 工具，設定 subagent_type="security-auditor"
- 提示：「對以下目標執行全面的安全評估：$ARGUMENTS。使用 Semgrep/SonarQube 執行 SAST 分析、使用 OWASP ZAP 進行 DAST 掃描、使用 Snyk/Trivy 進行依賴項稽核、使用 GitLeaks/TruffleHog 進行機密資料檢測。生成 SBOM 以進行供應鏈分析。識別 OWASP Top 10 漏洞、CWE 弱點和 CVE 曝露。」
- 輸出：詳細的漏洞報告，包含 CVSS 評分、可利用性分析、攻擊面映射、機密資料曝露報告、SBOM 清單
- 上下文：作為所有修復工作的初始基準

### 2. 威脅建模與風險分析
- 使用 Task 工具，設定 subagent_type="security-auditor"
- 提示：「使用 STRIDE 方法論對以下目標進行威脅建模：$ARGUMENTS。分析攻擊向量、創建攻擊樹、評估已識別漏洞的業務影響。將威脅映射到 MITRE ATT&CK 框架。根據可能性和影響對風險進行優先排序。」
- 輸出：威脅模型圖表、具有優先排序漏洞的風險矩陣、攻擊場景文件、業務影響分析
- 上下文：使用漏洞掃描結果來判定威脅優先順序

### 3. 架構安全審查
- 使用 Task 工具，設定 subagent_type="backend-api-security::backend-architect"
- 提示：「審查以下目標的架構安全弱點：$ARGUMENTS。評估服務邊界、資料流安全性、身份驗證/授權架構、加密實作、網路分段。設計零信任架構模式。參考威脅模型和漏洞發現。」
- 輸出：安全架構評估、零信任設計建議、服務網格安全需求、資料分類矩陣
- 上下文：整合威脅模型以解決架構漏洞

## 階段 2：漏洞修復

### 4. 關鍵漏洞修復
- 使用 Task 工具，設定 subagent_type="security-auditor"
- 提示：「協調立即修復以下目標中的關鍵漏洞（CVSS 7+）：$ARGUMENTS。使用參數化查詢修復 SQL 注入、使用輸出編碼修復 XSS、使用安全的工作階段管理修復身份驗證繞過、使用輸入驗證修復不安全的反序列化。為 CVE 應用安全修補程式。」
- 輸出：包含漏洞修復的修補程式碼、安全修補文件、回歸測試需求
- 上下文：處理漏洞評估中的高優先順序項目

### 5. 後端安全加固
- 使用 Task 工具，設定 subagent_type="backend-api-security::backend-security-coder"
- 提示：「為以下目標實施全面的後端安全控制：$ARGUMENTS。使用 OWASP ESAPI 新增輸入驗證、實施速率限制和 DDoS 保護、使用 OAuth2/JWT 驗證保護 API 端點、使用 AES-256/TLS 1.3 為靜態/傳輸中的資料新增加密。實施不暴露 PII 的安全日誌記錄。」
- 輸出：加固的 API 端點、驗證中介軟體、加密實作、安全配置範本
- 上下文：在漏洞修復基礎上建立預防性控制

### 6. 前端安全實作
- 使用 Task 工具，設定 subagent_type="frontend-mobile-security::frontend-security-coder"
- 提示：「為以下目標實施前端安全措施：$ARGUMENTS。配置基於 nonce 的 CSP 標頭策略、使用 DOMPurify 實施 XSS 防護、使用 PKCE OAuth2 實施安全的身份驗證流程、為外部資源新增 SRI、使用 SameSite/HttpOnly/Secure 標誌實施安全的 cookie 處理。」
- 輸出：安全的前端元件、CSP 策略配置、身份驗證流程實作、安全標頭配置
- 上下文：透過客戶端保護補充後端安全

### 7. 行動裝置安全加固
- 使用 Task 工具，設定 subagent_type="frontend-mobile-security::mobile-security-coder"
- 提示：「為以下目標實施行動應用程式安全：$ARGUMENTS。新增憑證固定、實施生物特徵驗證、使用加密保護本地儲存、使用 ProGuard/R8 混淆程式碼、實施防篡改和 root/越獄檢測、保護 IPC 通訊。」
- 輸出：加固的行動應用程式、安全配置檔案、混淆規則、憑證固定實作
- 上下文：將安全性擴展到行動平台（如適用）

## 階段 3：安全控制實施

### 8. 身份驗證與授權增強
- 使用 Task 工具，設定 subagent_type="security-auditor"
- 提示：「為以下目標實施現代化的身份驗證系統：$ARGUMENTS。部署具有 PKCE 的 OAuth2/OIDC、使用 TOTP/WebAuthn/FIDO2 實施 MFA、新增基於風險的身份驗證、使用最小權限原則實施 RBAC/ABAC、新增具有安全權杖輪換的工作階段管理。」
- 輸出：身份驗證服務配置、MFA 實作、授權策略、工作階段管理系統
- 上下文：基於架構審查強化存取控制

### 9. 基礎設施安全控制
- 使用 Task 工具，設定 subagent_type="deployment-strategies::deployment-engineer"
- 提示：「為以下目標部署基礎設施安全控制：$ARGUMENTS。配置 OWASP 保護的 WAF 規則、使用微分段實施網路分段、部署 IDS/IPS 系統、配置雲端安全群組和 NACL、使用速率限制和地理封鎖實施 DDoS 保護。」
- 輸出：WAF 配置、網路安全策略、IDS/IPS 規則、雲端安全配置
- 上下文：實施網路層級防禦

### 10. 機密資料管理實作
- 使用 Task 工具，設定 subagent_type="deployment-strategies::deployment-engineer"
- 提示：「為以下目標實施企業級機密資料管理：$ARGUMENTS。部署 HashiCorp Vault 或 AWS Secrets Manager、實施機密資料輪換策略、移除硬編碼的機密資料、配置最小權限的 IAM 角色、實施具有 HSM 支援的加密金鑰管理。」
- 輸出：機密資料管理配置、輪換策略、IAM 角色定義、金鑰管理程序
- 上下文：消除機密資料曝露漏洞

## 階段 4：驗證與合規性

### 11. 滲透測試與驗證
- 使用 Task 工具，設定 subagent_type="security-auditor"
- 提示：「對以下目標執行全面的滲透測試：$ARGUMENTS。執行已驗證和未驗證的測試、API 安全測試、業務邏輯測試、權限提升嘗試。使用 Burp Suite、Metasploit 和自訂漏洞利用。驗證所有安全控制的有效性。」
- 輸出：滲透測試報告、概念驗證漏洞利用、修復驗證、安全控制有效性指標
- 上下文：驗證所有已實施的安全措施

### 12. 合規性與標準驗證
- 使用 Task 工具，設定 subagent_type="security-auditor"
- 提示：「驗證以下目標是否符合安全框架：$ARGUMENTS。根據 OWASP ASVS Level 2、CIS Benchmarks、SOC2 Type II 要求、GDPR/CCPA 隱私控制、HIPAA/PCI-DSS（如適用）進行驗證。生成合規性證明報告。」
- 輸出：合規性評估報告、差距分析、修復需求、稽核證據收集
- 上下文：確保符合法規和產業標準

### 13. 安全監控與 SIEM 整合
- 使用 Task 工具，設定 subagent_type="incident-response::devops-troubleshooter"
- 提示：「為以下目標實施安全監控和 SIEM：$ARGUMENTS。部署 Splunk/ELK/Sentinel 整合、配置安全事件關聯、實施行為分析以進行異常檢測、設定自動化事件回應手冊、創建安全儀表板和警報。」
- 輸出：SIEM 配置、關聯規則、事件回應手冊、安全儀表板、警報定義
- 上下文：建立持續性安全監控

## 配置選項
- scanning_depth: "quick" | "standard" | "comprehensive"（預設：comprehensive）
- compliance_frameworks: ["OWASP", "CIS", "SOC2", "GDPR", "HIPAA", "PCI-DSS"]
- remediation_priority: "cvss_score" | "exploitability" | "business_impact"
- monitoring_integration: "splunk" | "elastic" | "sentinel" | "custom"
- authentication_methods: ["oauth2", "saml", "mfa", "biometric", "passwordless"]

## 成功標準
- 所有關鍵漏洞（CVSS 7+）已修復
- OWASP Top 10 漏洞已處理
- 滲透測試中零高風險發現
- 合規性框架驗證通過
- 安全監控能檢測並警示威脅
- 關鍵警報的事件回應時間 < 15 分鐘
- SBOM 已生成且漏洞已追蹤
- 所有機密資料透過安全保管庫管理
- 身份驗證實施 MFA 和安全的工作階段管理
- 安全測試已整合至 CI/CD 管道

## 協調說明
- 每個階段提供詳細的發現，為後續階段提供資訊
- Security-auditor 代理與特定領域代理協調進行修復
- 所有程式碼變更在實施前須經過安全審查
- 評估與修復之間的持續回饋循環
- 安全發現在集中式漏洞管理系統中追蹤
- 實施後定期安排安全審查

安全加固目標：$ARGUMENTS