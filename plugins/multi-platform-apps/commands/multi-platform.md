# 多平台功能開發工作流程

使用 API 優先架構和並行實作策略，在網頁、行動裝置和桌面平台上一致地建構和部署相同功能。

[延伸思考：此工作流程協調多個專業代理，以確保跨平台的功能一致性，同時維持平台特定的最佳化。協調策略強調共用合約和並行開發，並設定定期同步點。透過預先建立 API 合約和資料模型，團隊可以獨立工作，同時確保一致性。此工作流程的優點包括更快的上市時間、減少整合問題，以及可維護的跨平台程式碼庫。]

## 階段 1：架構與 API 設計（循序執行）

### 1. 定義功能需求與 API 合約
- 使用 Task 工具，設定 subagent_type="backend-architect"
- 提示詞："為功能設計 API 合約：$ARGUMENTS。建立 OpenAPI 3.1 規範，包含：
  - 具有正確 HTTP 方法和狀態碼的 RESTful 端點
  - GraphQL 架構（如適用於複雜資料查詢）
  - 即時功能的 WebSocket 事件
  - 具有驗證規則的請求/回應架構
  - 認證與授權需求
  - 速率限制與快取策略
  - 錯誤回應格式與代碼
  定義所有平台將使用的共用資料模型。"
- 預期輸出：完整的 API 規範、資料模型和整合指南

### 2. 設計系統與 UI/UX 一致性
- 使用 Task 工具，設定 subagent_type="ui-ux-designer"
- 提示詞："使用 API 規範建立跨平台設計系統：[前一步驟輸出]。包含：
  - 各平台的元件規範（Material Design、iOS HIG、Fluent）
  - 網頁響應式佈局（行動優先方法）
  - iOS（SwiftUI）和 Android（Material You）的原生模式
  - 桌面特定考量（鍵盤快捷鍵、視窗管理）
  - 無障礙需求（WCAG 2.2 Level AA）
  - 深色/淺色主題規範
  - 動畫與轉場指南"
- 來自前一步驟的上下文：API 端點、資料結構、認證流程
- 預期輸出：設計系統文件、元件庫規範、平台指南

### 3. 共用業務邏輯架構
- 使用 Task 工具，設定 subagent_type="comprehensive-review::architect-review"
- 提示詞："設計跨平台功能的共用業務邏輯架構。定義：
  - 核心領域模型與實體（平台無關）
  - 業務規則與驗證邏輯
  - 狀態管理模式（MVI/Redux/BLoC）
  - 快取與離線策略
  - 錯誤處理與重試政策
  - 平台特定的適配器模式
  考慮使用 Kotlin Multiplatform 用於行動裝置，或 TypeScript 用於網頁/桌面共用。"
- 來自前一步驟的上下文：API 合約、資料模型、UI 需求
- 預期輸出：共用程式碼架構、平台抽象層、實作指南

## 階段 2：並行平台實作

### 4a. 網頁實作（React/Next.js）
- 使用 Task 工具，設定 subagent_type="frontend-developer"
- 提示詞："使用以下技術實作網頁版本功能：
  - React 18+ 搭配 Next.js 14+ App Router
  - TypeScript 以確保型別安全
  - TanStack Query 用於 API 整合：[API 規範]
  - Zustand/Redux Toolkit 用於狀態管理
  - Tailwind CSS 搭配設計系統：[設計規範]
  - Progressive Web App 功能
  - 適當的 SSR/SSG 最佳化
  - Web vitals 最佳化（LCP < 2.5s，FID < 100ms）
  遵循共用業務邏輯：[架構文件]"
- 來自前一步驟的上下文：API 合約、設計系統、共用邏輯模式
- 預期輸出：完整的網頁實作與測試

### 4b. iOS 實作（SwiftUI）
- 使用 Task 工具，設定 subagent_type="ios-developer"
- 提示詞："使用以下技術實作 iOS 版本：
  - SwiftUI 搭配 iOS 17+ 功能
  - Swift 5.9+ 搭配 async/await
  - URLSession 搭配 Combine 用於 API：[API 規範]
  - Core Data/SwiftData 用於持久化
  - 遵循設計系統：[iOS HIG 規範]
  - Widget 擴充功能（如適用）
  - 平台特定功能（Face ID、Haptics、Live Activities）
  - 可測試的 MVVM 架構
  遵循共用模式：[架構文件]"
- 來自前一步驟的上下文：API 合約、iOS 設計指南、共用模型
- 預期輸出：原生 iOS 實作與單元/UI 測試

### 4c. Android 實作（Kotlin/Compose）
- 使用 Task 工具，設定 subagent_type="mobile-developer"
- 提示詞："使用以下技術實作 Android 版本：
  - Jetpack Compose 搭配 Material 3
  - Kotlin coroutines 與 Flow
  - Retrofit/Ktor 用於 API：[API 規範]
  - Room database 用於本地儲存
  - Hilt 用於依賴注入
  - Material You 動態主題：[設計規範]
  - 平台功能（生物辨識認證、widgets）
  - Clean architecture 搭配 MVI 模式
  遵循共用邏輯：[架構文件]"
- 來自前一步驟的上下文：API 合約、Material Design 規範、共用模式
- 預期輸出：原生 Android 實作與測試

### 4d. 桌面實作（選用 - Electron/Tauri）
- 使用 Task 工具，設定 subagent_type="frontend-mobile-development::frontend-developer"
- 提示詞："使用 Tauri 2.0 或 Electron 實作桌面版本：
  - 盡可能共用網頁程式碼庫
  - 原生作業系統整合（系統托盤、通知）
  - 檔案系統存取（如需要）
  - 自動更新功能
  - 程式碼簽署與公證設定
  - 鍵盤快捷鍵與選單列
  - 多視窗支援（如適用）
  重用網頁元件：[網頁實作]"
- 來自前一步驟的上下文：網頁實作、桌面特定需求
- 預期輸出：桌面應用程式與平台套件

## 階段 3：整合與驗證

### 5. API 文件與測試
- 使用 Task 工具，設定 subagent_type="documentation-generation::api-documenter"
- 提示詞："建立完整的 API 文件，包含：
  - 互動式 OpenAPI/Swagger 文件
  - 平台特定的整合指南
  - 各平台的 SDK 範例
  - 認證流程圖
  - 速率限制與配額資訊
  - Postman/Insomnia collections
  - WebSocket 連線範例
  - 錯誤處理最佳實務
  - API 版本控制策略
  使用平台實作測試所有端點。"
- 來自前一步驟的上下文：已實作的平台、API 使用模式
- 預期輸出：完整的 API 文件入口、測試結果

### 6. 跨平台測試與功能一致性
- 使用 Task 工具，設定 subagent_type="unit-testing::test-automator"
- 提示詞："驗證所有平台的功能一致性：
  - 功能測試矩陣（功能運作完全相同）
  - UI 一致性驗證（遵循設計系統）
  - 各平台的效能基準
  - 無障礙測試（平台特定工具）
  - 網路彈性測試（離線、慢速連線）
  - 資料同步驗證
  - 平台特定的邊緣案例
  - 端對端使用者旅程測試
  建立測試報告，包含任何平台差異。"
- 來自前一步驟的上下文：所有平台實作、API 文件
- 預期輸出：測試報告、一致性矩陣、效能指標

### 7. 平台特定最佳化
- 使用 Task 工具，設定 subagent_type="application-performance::performance-engineer"
- 提示詞："最佳化各平台實作：
  - 網頁：Bundle 大小、延遲載入、CDN 設定、SEO
  - iOS：應用程式大小、啟動時間、記憶體使用、電池
  - Android：APK 大小、啟動時間、幀率、電池
  - 桌面：二進位檔大小、資源使用、啟動時間
  - API：回應時間、快取、壓縮
  在維持功能一致性的同時，充分利用平台優勢。
  記錄最佳化技術與取捨。"
- 來自前一步驟的上下文：測試結果、效能指標
- 預期輸出：最佳化的實作、效能改善

## 設定選項

- **--platforms**：指定目標平台（web,ios,android,desktop）
- **--api-first**：在 UI 實作前先產生 API（預設：true）
- **--shared-code**：使用 Kotlin Multiplatform 或類似技術（預設：評估）
- **--design-system**：使用現有或建立新的（預設：建立）
- **--testing-strategy**：單元、整合、端對端測試（預設：全部）

## 成功標準

- 在實作前定義並驗證 API 合約
- 所有平台達成功能一致性，差異 <5%
- 效能指標符合平台特定標準
- 符合無障礙標準（至少 WCAG 2.2 AA）
- 跨平台測試顯示一致行為
- 所有平台的文件完整
- 平台間程式碼重用率 >40%（如適用）
- 使用者體驗針對各平台慣例進行最佳化

## 平台特定考量

**網頁**：PWA 功能、SEO 最佳化、瀏覽器相容性
**iOS**：App Store 指南、TestFlight 發布、iOS 特定功能
**Android**：Play Store 需求、Android App Bundles、裝置碎片化
**桌面**：程式碼簽署、自動更新、作業系統特定安裝程式

初始功能規範：$ARGUMENTS