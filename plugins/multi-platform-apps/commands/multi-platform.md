# 多平台功能開發工作流程

使用 API 優先架構和平行實作策略，在網頁、行動裝置和桌面平台上一致地建置和部署相同的功能。

[擴展思考：此工作流程協調多個專業代理，以確保跨平台的功能一致性，同時維持平台特定的最佳化。協調策略強調共享契約和平行開發，並定期進行同步點。透過預先建立 API 契約和資料模型，團隊可以獨立工作，同時確保一致性。工作流程的好處包括更快的上市時間、減少整合問題，以及可維護的跨平台程式碼庫。]

## 階段 1：架構和 API 設計（循序）

### 1. 定義功能需求和 API 契約
- 使用 Task 工具，subagent_type="backend-architect"
- 提示："為功能設計 API 契約：$ARGUMENTS。建立 OpenAPI 3.1 規格，包含：
  - 具備適當 HTTP 方法和狀態碼的 RESTful 端點
  - GraphQL schema（如適用於複雜資料查詢）
  - 即時功能的 WebSocket 事件
  - 具備驗證規則的請求/回應 schemas
  - 認證和授權需求
  - 速率限制和快取策略
  - 錯誤回應格式和代碼
  定義所有平台將使用的共享資料模型。"
- 預期輸出：完整的 API 規格、資料模型和整合指南

### 2. 設計系統和 UI/UX 一致性
- 使用 Task 工具，subagent_type="ui-ux-designer"
- 提示："使用 API 規格為功能建立跨平台設計系統：[先前輸出]。包含：
  - 每個平台的元件規格（Material Design、iOS HIG、Fluent）
  - 網頁的響應式版面配置（行動優先方法）
  - iOS（SwiftUI）和 Android（Material You）的原生模式
  - 桌面特定考量（鍵盤快速鍵、視窗管理）
  - 無障礙需求（WCAG 2.2 Level AA）
  - 深色/淺色主題規格
  - 動畫和轉場指南"
- 來自先前的上下文：API 端點、資料結構、認證流程
- 預期輸出：設計系統文件、元件庫規格、平台指南

### 3. 共享商業邏輯架構
- 使用 Task 工具，subagent_type="comprehensive-review::architect-review"
- 提示："為跨平台功能設計共享商業邏輯架構。定義：
  - 核心領域模型和實體（平台無關）
  - 商業規則和驗證邏輯
  - 狀態管理模式（MVI/Redux/BLoC）
  - 快取和離線策略
  - 錯誤處理和重試政策
  - 平台特定的適配器模式
  考慮行動裝置使用 Kotlin Multiplatform 或網頁/桌面共享使用 TypeScript。"
- 來自先前的上下文：API 契約、資料模型、UI 需求
- 預期輸出：共享程式碼架構、平台抽象層、實作指南

## 階段 2：平行平台實作

### 4a. 網頁實作（React/Next.js）
- 使用 Task 工具，subagent_type="frontend-developer"
- 提示："使用以下技術實作網頁版本的功能：
  - React 18+ 具備 Next.js 14+ App Router
  - TypeScript 用於型別安全
  - TanStack Query 用於 API 整合：[API 規格]
  - Zustand/Redux Toolkit 用於狀態管理
  - Tailwind CSS 具備設計系統：[設計規格]
  - Progressive Web App 功能
  - 適當的 SSR/SSG 最佳化
  - Web vitals 最佳化（LCP < 2.5s、FID < 100ms）
  遵循共享商業邏輯：[架構文件]"
- 來自先前的上下文：API 契約、設計系統、共享邏輯模式
- 預期輸出：完整的網頁實作，包含測試

### 4b. iOS 實作（SwiftUI）
- 使用 Task 工具，subagent_type="ios-developer"
- 提示："使用以下技術實作 iOS 版本：
  - SwiftUI 具備 iOS 17+ 功能
  - Swift 5.9+ 具備 async/await
  - URLSession 具備 Combine 用於 API：[API 規格]
  - Core Data/SwiftData 用於持久化
  - 設計系統合規：[iOS HIG 規格]
  - Widget 擴充功能（如適用）
  - 平台特定功能（Face ID、Haptics、Live Activities）
  - 可測試的 MVVM 架構
  遵循共享模式：[架構文件]"
- 來自先前的上下文：API 契約、iOS 設計指南、共享模型
- 預期輸出：原生 iOS 實作，包含單元/UI 測試

### 4c. Android 實作（Kotlin/Compose）
- 使用 Task 工具，subagent_type="mobile-developer"
- 提示："使用以下技術實作 Android 版本：
  - Jetpack Compose 具備 Material 3
  - Kotlin coroutines 和 Flow
  - Retrofit/Ktor 用於 API：[API 規格]
  - Room 資料庫用於本地儲存
  - Hilt 用於依賴注入
  - Material You 動態主題：[設計規格]
  - 平台功能（生物辨識認證、widgets）
  - 具備 MVI 模式的 Clean architecture
  遵循共享邏輯：[架構文件]"
- 來自先前的上下文：API 契約、Material Design 規格、共享模式
- 預期輸出：原生 Android 實作，包含測試

### 4d. 桌面實作（選擇性 - Electron/Tauri）
- 使用 Task 工具，subagent_type="frontend-mobile-development::frontend-developer"
- 提示："使用 Tauri 2.0 或 Electron 實作桌面版本，包含：
  - 盡可能共享網頁程式碼庫
  - 原生 OS 整合（系統托盤、通知）
  - 檔案系統存取（如需要）
  - 自動更新功能
  - 程式碼簽署和公證設定
  - 鍵盤快速鍵和選單列
  - 多視窗支援（如適用）
  重用網頁元件：[網頁實作]"
- 來自先前的上下文：網頁實作、桌面特定需求
- 預期輸出：桌面應用程式，包含平台套件

## 階段 3：整合和驗證

### 5. API 文件和測試
- 使用 Task 工具，subagent_type="documentation-generation::api-documenter"
- 提示："建立全方位的 API 文件，包含：
  - 互動式 OpenAPI/Swagger 文件
  - 平台特定的整合指南
  - 每個平台的 SDK 範例
  - 認證流程圖
  - 速率限制和配額資訊
  - Postman/Insomnia 集合
  - WebSocket 連線範例
  - 錯誤處理最佳實務
  - API 版本控制策略
  測試所有端點與平台實作。"
- 來自先前的上下文：已實作的平台、API 使用模式
- 預期輸出：完整的 API 文件入口網站、測試結果

### 6. 跨平台測試和功能一致性
- 使用 Task 工具，subagent_type="unit-testing::test-automator"
- 提示："驗證所有平台的功能一致性：
  - 功能測試矩陣（功能運作一致）
  - UI 一致性驗證（遵循設計系統）
  - 每個平台的效能基準
  - 無障礙測試（平台特定工具）
  - 網路韌性測試（離線、慢速連線）
  - 資料同步驗證
  - 平台特定的邊緣案例
  - 端對端使用者旅程測試
  建立測試報告，包含任何平台差異。"
- 來自先前的上下文：所有平台實作、API 文件
- 預期輸出：測試報告、一致性矩陣、效能指標

### 7. 平台特定最佳化
- 使用 Task 工具，subagent_type="application-performance::performance-engineer"
- 提示："最佳化每個平台實作：
  - 網頁：套件大小、延遲載入、CDN 設定、SEO
  - iOS：應用程式大小、啟動時間、記憶體使用、電池
  - Android：APK 大小、啟動時間、影格率、電池
  - 桌面：二進位大小、資源使用、啟動時間
  - API：回應時間、快取、壓縮
  在利用平台優勢的同時維持功能一致性。
  記錄最佳化技術和取捨。"
- 來自先前的上下文：測試結果、效能指標
- 預期輸出：最佳化的實作、效能改進

## 配置選項

- **--platforms**：指定目標平台（web,ios,android,desktop）
- **--api-first**：在 UI 實作前生成 API（預設：true）
- **--shared-code**：使用 Kotlin Multiplatform 或類似技術（預設：評估）
- **--design-system**：使用現有或建立新的（預設：建立）
- **--testing-strategy**：Unit、integration、e2e（預設：all）

## 成功標準

- 在實作前定義和驗證 API 契約
- 所有平台達成功能一致性，差異 <5%
- 效能指標符合平台特定標準
- 符合無障礙標準（最低 WCAG 2.2 AA）
- 跨平台測試顯示一致的行為
- 所有平台的文件完整
- 平台間的程式碼重用 >40%（如適用）
- 針對每個平台的慣例最佳化使用者體驗

## 平台特定考量

**網頁**：PWA 功能、SEO 最佳化、瀏覽器相容性
**iOS**：App Store 指南、TestFlight 發佈、iOS 特定功能
**Android**：Play Store 需求、Android App Bundles、裝置碎片化
**桌面**：程式碼簽署、自動更新、OS 特定安裝程式

初始功能規格：$ARGUMENTS
