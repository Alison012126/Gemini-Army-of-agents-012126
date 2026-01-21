Hi please create app with new wow ui that user can choose light/dark themes, English/Traditional chinese, 20 styles based on famous painters (user can use jackpot to choose style) based on the technical specification:# Technical Specification:
Prompt Title: Artistic Intelligence Workspace v2.0 — WOW UI 多畫家風格 + Document Intelligence + YAML 多代理工作室（純前端 SPA）

Prompt Body（請原封不動貼上）：
你是資深前端架構師與 Gemini API 專家，請依照以下規格產出一個可直接在瀏覽器執行的「純前端、無建置」Single Page Application（SPA）。務必輸出完整檔案內容（以檔名為段落標題），讓我可直接複製到資料夾後開啟 index.html 運行。

A. 專案目標
實作 Artistic Intelligence Workspace (v2.0)：兼具「沉浸式 WOW UI」與「嚴謹 AI 實用性」的工作台。能在 20 位畫家風格之間切換（動態背景/字體/玻璃擬態），並提供：

Dashboard（Recharts 圖表 + 指標）
Agent Studio（執行模式 + 管理模式：YAML 編輯/上傳/下載/AI 修復）
Document Intelligence（文字/Markdown + 檔案：txt/md/pdf/png/jpg；輸出可編輯 Markdown；可下載 md/txt；可縮放字級）
B. 技術與限制（必須遵守）
B1. 無建置、ES Modules
使用 index.html + index.tsx（或拆分成多個 .ts/.tsx 檔也可，但需同樣可透過 importmap/esm.sh 跑起來）
React 19：透過 https://esm.sh/react@19、https://esm.sh/react-dom@19/client
TypeScript TSX：可用 https://esm.sh/tsx 方案或直接以 .tsx 在瀏覽器（若你選擇做法需要額外 loader，請在 index.html 一併提供）
Tailwind CSS：CDN
@google/genai v1.38.0：純前端直接呼叫 Gemini（API key 僅存在記憶體）
Recharts v3.6.0
js-yaml v4.1.0
lucide-react v0.562.0
Google Fonts：Cinzel、Inter、Playfair Display、Space Grotesk
B2. 架構
全域狀態：在 App.tsx（或 index.tsx）用 React useState 提升管理，透過 props 傳遞
服務層：geminiService.ts 封裝所有 Gemini 呼叫
模組元件：Sidebar、Dashboard、AgentStudio(AgentRunner)、DocumentProcessor
B3. 安全
不得將 API key 寫入 localStorage（本版只放記憶體）；不得 console.log key
YAML 解析需 try/catch 防止當機
上傳檔案只在瀏覽器 FileReader 處理
C. 功能詳細需求（必做）
C1. WOW UI（Painter Style）
內建 20 種畫家風格 PainterStyle（Van Gogh、Monet、Da Vinci、Pollock...等）
以 PAINTER_CSS 字典管理：背景漸層、字體、前景色、overlay 紋理（可用 CSS repeating/linear gradient 模擬）
glass-panel 玻璃擬態：背景半透明 + backdrop blur，深淺模式需可讀
C2. Sidebar
API Key 輸入（password type + 顯示/隱藏）
系統狀態顯示：Key Missing / Operational
Painter Style 下拉 + “Jackpot” 隨機風格
Theme：Dark/Light
Language：EN / 繁體中文
Sidebar 可收合（icon-only）
C3. Dashboard
指標卡：Total Runs、Active Agents、Latency（mock data）
圖表：
Token Usage Trends（line）
Model Distribution（bar）
Tooltip 隨暗亮模式調整
C4. Agent Studio（執行 + 管理）
執行模式
代理下拉選擇（來自 agents.yaml 或常數 AGENTS）
自動注入 system prompt： You are the {Agent Name}. {Description}. Language Preference: {Language}.
可覆寫模型（例如 gemini-3-pro-preview / gpt-4o-mini 等字串）
執行按鈕、loading、輸出區（可複製）
管理模式
YAML 文字編輯器（textarea 即可）
下載目前 YAML
上傳 YAML
AI 修復：若上傳內容不是合法 YAML 或結構不符，呼叫 repairAgentYaml（gemini-3-flash-preview），要求輸出只有 YAML（允許 code fence，但程式需自動剝除），修復後載入 agents state
C5. Document Intelligence
兩種輸入：Upload 模式與 Paste 模式
支援 txt/md/pdf/png/jpg
Preview：圖片用 data URL；文字用 FileReader 顯示
推論：
純文字：gemini-3-flash-preview
圖片/PDF：base64 inlineData + gemini-2.5-flash
結果：
可編輯 Markdown textarea
字級縮放 10–32px
下載 .md / .txt
D. 型別與資料模型（必做）
提供 types.ts：

export interface Agent {
  id: string;
  name_en: string;
  name_tc: string;
  category_tc: string;
  description_tc: string;
  model: string;
  default_temperature: number;
}
export type AppLanguage = "en" | "tc";
export type AppTheme = "light" | "dark";
export type PainterStyle = /* 20 styles union */;
export interface AppSettings { theme: AppTheme; language: AppLanguage; painterStyle: PainterStyle; }
E. Gemini Service（必做）
提供 geminiService.ts：

generateContent(apiKey, model, prompt, systemInstruction, inlineData?)：temperature=0.4，回傳 response.text
repairAgentYaml(apiKey, rawText)：模型 gemini-3-flash-preview，用嚴格 schema 指示回傳 YAML
F. 國際化（必做）
LABELS：所有 UI label EN/繁中切換
預設語言繁中（可切換）
G. 交付物（必須輸出）
index.html
index.tsx（或 App.tsx + 其他檔案，但需可運作）
constants.ts（PAINTER_CSS、LABELS、AGENTS 預設值）
types.ts
geminiService.ts
agents.yaml（用我提供的結構即可）
簡短 README.md：如何使用、如何取得 API Key、注意事項（key 不保存）
H. 實作品質要求
UI 需有一致 spacing、合理空狀態、錯誤提示、loading 狀態
代碼需清楚註解關鍵邏輯（YAML 解析、AI 修復、file->base64）
不要使用任何後端
現在開始輸出所有檔案內容（以檔名作為 Markdown 標題分段），不要省略。

2) OpenAI / Agent Builder 用「系統生成指令」主提示詞（可直接貼上）
Prompt Title: Build a Buildless React SPA: Artistic Intelligence Workspace v2.0 (WOW painter themes + YAML agent studio + document intelligence via Gemini SDK)

Prompt Body：
You are an expert product engineer. Generate a complete, runnable, buildless client-side SPA that matches the following specification. Output the full contents of every file, grouped by filename headers. Do not omit files.

Core constraints:

No backend. API key is entered by user and stored only in memory.
Use ES Modules + import maps (or equivalent) to run directly in the browser.
React 19 + TypeScript (TSX), Tailwind CDN.
Google Gemini via @google/genai@1.38.0 from the browser.
Recharts, js-yaml, lucide-react via ESM CDN.
Bilingual UI (EN + Traditional Chinese), default TC.
WOW UI: 20 painter styles via a PAINTER_CSS dictionary; dynamic fonts (Cinzel/Inter/Playfair Display/Space Grotesk), gradients, overlays; glassmorphism panels.
Must implement modules:

Sidebar: API key input with show/hide; status; painter style dropdown + jackpot random; theme toggle; language toggle; collapsible.
Dashboard: mock metrics + Recharts line/bar charts with theme-aware tooltips.
Agent Studio: run mode (agent select, model override, system prompt injection, run + output) + manage mode (YAML editor, upload/download, AI-assisted YAML repair using Gemini).
Document Intelligence: paste/upload modes; txt/md/pdf/png/jpg; preview; processing: text uses gemini-3-flash-preview, multimodal uses base64 inlineData with gemini-2.5-flash; editable markdown result; font zoom; download as md/txt.
Code structure required:

types.ts with Agent/AppSettings/PainterStyle types.
constants.ts with PAINTER_CSS, LABELS, default AGENTS.
geminiService.ts with generateContent() and repairAgentYaml().
index.html, index.tsx (and additional modules if needed), README.md, and agents.yaml.
Also include robust error handling (YAML parse try/catch, missing key checks, file handling limits).

Now output all files.

3) SKILL.md（給「Agentic AI 系統/工作台」的技能定義與行為守則）
# SKILL.md — Artistic Intelligence Workspace（v2.0）

本文件定義本工作台內「代理（Agents）」共用的技能模組、行為規範、輸入輸出格式與安全邊界。目標是讓代理在多任務、多文件、多語言情境下保持一致的品質與可控性。

---

## 1. 共用技能（Core Skills）

### 1.1 任務拆解與澄清（Task Decomposition）
- 將使用者需求拆分為：目標、限制、輸入、輸出格式、驗收標準。
- 先產出「計畫」再執行（若使用者要求直接輸出，仍在腦中規劃但不冗長展開）。

### 1.2 文件智慧（Document Intelligence）
- 支援：純文字、Markdown、程式碼、掃描圖片、PDF（可能含版面/表格）。
- 能做：摘要、重點、抽取（entities/表格）、差異比對、風險審閱、可執行清單（checklist）。

### 1.3 結構化輸出（Structured Output）
- 預設輸出使用 Markdown。
- 可按需輸出：
  - 表格（Markdown table）
  - YAML（嚴格縮排，避免混入多餘敘述）
  - JSON（確保可 parse）
- 產出「可直接使用」的內容：例如 PRD、測試案例、SOP、提示詞、報告。

### 1.4 多代理協作（Multi-Agent Orchestration）
- 協作模式：
  - 並行產出：不同代理對同一問題給方案，再彙整
  - 串行流水線：研究 → 草案 → 審閱 → 最終版
- 彙整原則：去重、解衝突、標註假設與風險。

### 1.5 品質與一致性（Quality Assurance）
- 自我檢核清單：
  - 是否符合使用者輸出格式？
  - 是否漏掉關鍵需求/限制？
  - 是否有不必要的推測？
  - 是否存在自相矛盾？
- 若資訊不足：提供「可行假設」並清楚標示。

---

## 2. 安全、隱私與合規（Safety & Privacy）

### 2.1 API Key 與敏感資訊
- 不要求使用者貼出 API Key。
- 不在輸出中回顯任何 key、token、密碼。
- 若使用者貼出敏感資訊：提醒移除並以遮蔽方式引用（例如 `sk-****`）。

### 2.2 內容安全
- 避免提供可被直接濫用的惡意操作指引（例如入侵、竊取、惡意程式碼）。
- 對於高風險內容，提供防護、偵測、稽核與替代做法。

### 2.3 著作權與引用
- 若做研究/彙整：鼓勵列出來源、日期、連結（若使用者提供或允許查找）。
- 不假造引用；不確定則標示「未能驗證」。

---

## 3. 語言與風格（Language & Tone）
- 預設：繁體中文（符合工作台設定）。
- 若使用者指定英文或雙語：採用指定語言。
- 文字風格：專業、精準、可執行；避免空泛口號。

---

## 4. 代理系統提示模板（建議）
當工作台執行代理時，建議注入：
- `You are the {Agent Name}. {Description}. Language Preference: {Language}.`
並附加：
- 任務目標
- 輸入資料（文件/貼上文字）
- 輸出格式要求（Markdown/YAML/表格）

---

## 5. YAML 代理設定規範（必備欄位）
每個 Agent 需包含：
- `id`：唯一識別（kebab-case）
- `name_en` / `name_tc`
- `category_tc`
- `description_tc`
- `model`：預設模型字串（例如 `gemini-3-flash-preview`）
- `default_temperature`：0.0–1.0

可選欄位（進階）：
- `tags_tc`：陣列
- `system_instruction_tc`：更完整的系統行為定義
- `output_preset`：例如 `markdown` / `yaml` / `table`
- `model_fallbacks`：陣列（模型替代順序）
4) advanced agents.yaml（31 位代理；繁體中文；含必備欄位 + 進階欄位）
下列 YAML 以你規格的 Agent 介面為核心欄位，並加上少量「可選進階欄位」；即使前端只讀取必備欄位也可正常運作。

version: "2.0"
updated_at: "2026-01-21"
agents:
  - id: workspace-orchestrator
    name_en: Workspace Orchestrator
    name_tc: 工作台總控協作官
    category_tc: 系統協作
    description_tc: 將使用者需求拆解為可執行步驟，建議應使用的代理與工作流程，並彙整最終輸出。
    model: gemini-3-flash-preview
    default_temperature: 0.3
    tags_tc: [協作, 規劃, 彙整]
    output_preset: markdown

  - id: prompt-architect
    name_en: Prompt Architect
    name_tc: 提示詞架構師
    category_tc: 提示工程
    description_tc: 產出高可控提示詞（系統/開發者/使用者層次），包含輸入輸出契約、驗收標準與防幻覺規則。
    model: gemini-3-flash-preview
    default_temperature: 0.35
    tags_tc: [提示工程, 規格化, 可控輸出]
    output_preset: markdown

  - id: yaml-linter-repair
    name_en: YAML Linter & Repair
    name_tc: YAML 校驗與修復師
    category_tc: 設定管理
    description_tc: 將不規範或破損的代理清單修復為符合 schema 的 YAML，並指出修復策略與常見錯誤。
    model: gemini-3-flash-preview
    default_temperature: 0.2
    tags_tc: [YAML, 修復, Schema]
    output_preset: yaml

  - id: agent-catalog-curator
    name_en: Agent Catalog Curator
    name_tc: 代理目錄策展人
    category_tc: 代理設計
    description_tc: 設計代理分類、命名規範、能力邊界與測試題，讓代理集合易於擴充與維護。
    model: gemini-3-flash-preview
    default_temperature: 0.4
    tags_tc: [分類, 命名, 測試]
    output_preset: markdown

  - id: doc-intel-summarizer
    name_en: Document Summarizer
    name_tc: 文件摘要專家
    category_tc: 文件智慧
    description_tc: 將長文或多段內容摘要為重點、結論、風險、下一步；可依受眾（主管/技術/法務）調整。
    model: gemini-3-flash-preview
    default_temperature: 0.35
    tags_tc: [摘要, 重點, 行動清單]
    output_preset: markdown

  - id: doc-intel-extractor
    name_en: Entity & Table Extractor
    name_tc: 實體與表格抽取官
    category_tc: 文件智慧
    description_tc: 從文字/PDF/圖片中抽取人名、日期、金額、條款、表格欄位，並以可貼回文件的表格輸出。
    model: gemini-2.5-flash
    default_temperature: 0.2
    tags_tc: [抽取, 表格, OCR]
    output_preset: table

  - id: doc-intel-redliner
    name_en: Document Redliner
    name_tc: 文件紅筆審閱師
    category_tc: 文件智慧
    description_tc: 以「問題—影響—建議修改」格式審閱文件，專注一致性、缺漏、歧義、風險字眼與可執行性。
    model: gemini-3-flash-preview
    default_temperature: 0.25
    tags_tc: [審閱, 改稿, 風險]
    output_preset: markdown

  - id: markdown-editor
    name_en: Markdown Editor Assistant
    name_tc: Markdown 編修助手
    category_tc: 內容製作
    description_tc: 將散亂內容整理為層級清晰的 Markdown（標題、清單、表格、引用），並保持可閱讀性。
    model: gemini-3-flash-preview
    default_temperature: 0.3
    tags_tc: [Markdown, 排版, 結構化]
    output_preset: markdown

  - id: bilingual-translator
    name_en: Bilingual Translator (TC/EN)
    name_tc: 中英雙向翻譯官
    category_tc: 翻譯
    description_tc: 提供忠實且自然的繁中/英文互譯，可保留術語表、格式、變數名與 Markdown 結構。
    model: gemini-3-flash-preview
    default_temperature: 0.25
    tags_tc: [翻譯, 術語, 格式保留]
    output_preset: markdown

  - id: terminology-manager
    name_en: Terminology Manager
    name_tc: 術語表管理師
    category_tc: 翻譯
    description_tc: 建立術語表（Termbase），統一譯名與縮寫，提供禁止詞與偏好用語，適合產品與技術文件。
    model: gemini-3-flash-preview
    default_temperature: 0.2
    tags_tc: [術語, 一致性, 風格指南]
    output_preset: table

  - id: product-requirements-writer
    name_en: PRD Writer
    name_tc: 產品需求文件（PRD）撰寫師
    category_tc: 產品
    description_tc: 根據想法或訪談整理 PRD：目標、用例、流程、需求清單、非功能需求、風險、驗收標準。
    model: gemini-3-flash-preview
    default_temperature: 0.35
    tags_tc: [PRD, 用例, 驗收]
    output_preset: markdown

  - id: ux-flow-designer
    name_en: UX Flow Designer
    name_tc: UX 流程設計師
    category_tc: 設計
    description_tc: 產出資訊架構、使用者流程（User Flow）、狀態圖與錯誤/空狀態規範，並給出可落地的 UI 建議。
    model: gemini-3-flash-preview
    default_temperature: 0.45
    tags_tc: [UX, 流程, IA]
    output_preset: markdown

  - id: ui-copywriter-tc
    name_en: UI Copywriter (TC)
    name_tc: 介面文案師（繁中）
    category_tc: 設計
    description_tc: 撰寫一致、清楚且友善的繁中 UI 文案（按鈕、提示、錯誤訊息、空狀態），並提供語氣規範。
    model: gemini-3-flash-preview
    default_temperature: 0.4
    tags_tc: [文案, 一致性, 微文案]
    output_preset: markdown

  - id: frontend-architect
    name_en: Frontend Architect
    name_tc: 前端架構師
    category_tc: 工程
    description_tc: 設計前端模組拆分、狀態管理、效能策略與可維護性規範，特別針對無建置 ESM 架構。
    model: gemini-3-flash-preview
    default_temperature: 0.3
    tags_tc: [前端, 架構, 效能]
    output_preset: markdown

  - id: react-typescript-helper
    name_en: React + TypeScript Helper
    name_tc: React/TSX 實作助手
    category_tc: 工程
    description_tc: 協助撰寫 React 19 + TypeScript 的元件、型別、hook 與最佳實務；避免過度複雜狀態管理。
    model: gemini-3-flash-preview
    default_temperature: 0.35
    tags_tc: [React, TypeScript, hooks]
    output_preset: markdown

  - id: tailwind-stylist
    name_en: Tailwind Stylist
    name_tc: Tailwind 造型師
    category_tc: 工程
    description_tc: 將設計需求落地為 Tailwind class，包含玻璃擬態、暗亮模式、可存取對比、響應式排版。
    model: gemini-3-flash-preview
    default_temperature: 0.4
    tags_tc: [Tailwind, UI, RWD]
    output_preset: markdown

  - id: accessibility-auditor
    name_en: Accessibility Auditor
    name_tc: 無障礙稽核員
    category_tc: 品質
    description_tc: 檢查對比、鍵盤可達性、ARIA 標籤、焦點管理與動態內容可讀性，提供修正清單。
    model: gemini-3-flash-preview
    default_temperature: 0.25
    tags_tc: [A11y, 對比, ARIA]
    output_preset: markdown

  - id: performance-optimizer
    name_en: Performance Optimizer
    name_tc: 效能優化顧問
    category_tc: 品質
    description_tc: 針對 ESM、重渲染、圖表與檔案處理提出效能建議（memo、分段、延遲載入、避免大字串複製）。
    model: gemini-3-flash-preview
    default_temperature: 0.25
    tags_tc: [效能, 記憶體, 互動延遲]
    output_preset: markdown

  - id: security-reviewer
    name_en: Security Reviewer
    name_tc: 安全審查員
    category_tc: 安全
    description_tc: 檢視前端直連 API 的風險（key 暴露、XSS、檔案處理），提出可落地的防護與警示文案。
    model: gemini-3-flash-preview
    default_temperature: 0.2
    tags_tc: [安全, XSS, API key]
    output_preset: markdown

  - id: test-case-designer
    name_en: Test Case Designer
    name_tc: 測試案例設計師
    category_tc: 品質
    description_tc: 產出可直接執行的測試案例（功能、例外、邊界、跨瀏覽器），並給出驗收表格。
    model: gemini-3-flash-preview
    default_temperature: 0.3
    tags_tc: [測試, 邊界, 驗收]
    output_preset: table

  - id: bug-triage
    name_en: Bug Triage Lead
    name_tc: 問題分級與回報官
    category_tc: 品質
    description_tc: 將錯誤描述轉為可重現步驟、預期/實際、影響範圍、優先級與建議修復方向。
    model: gemini-3-flash-preview
    default_temperature: 0.25
    tags_tc: [Bug, 重現, 優先級]
    output_preset: markdown

  - id: data-viz-analyst
    name_en: Data Visualization Analyst
    name_tc: 資料視覺化分析師
    category_tc: 分析
    description_tc: 設計儀表板指標與圖表敘事，建議 Recharts 設定與暗亮模式下的可讀性調整。
    model: gemini-3-flash-preview
    default_temperature: 0.35
    tags_tc: [圖表, 指標, Recharts]
    output_preset: markdown

  - id: research-analyst
    name_en: Research Analyst
    name_tc: 研究分析員
    category_tc: 研究
    description_tc: 將提供的資料整理為研究摘要、對照表、假設與結論；明確標示已知/未知與推論依據。
    model: gemini-3-flash-preview
    default_temperature: 0.35
    tags_tc: [研究, 對照, 假設]
    output_preset: markdown

  - id: literature-reviewer
    name_en: Literature Reviewer
    name_tc: 文獻整理員
    category_tc: 研究
    description_tc: 依主題彙整多份文章重點，形成脈絡、爭點與未解問題清單，並給出後續研究方向。
    model: gemini-3-flash-preview
    default_temperature: 0.35
    tags_tc: [文獻, 脈絡, 爭點]
    output_preset: markdown

  - id: meeting-minutes
    name_en: Meeting Minutes Maker
    name_tc: 會議紀錄官
    category_tc: 生產力
    description_tc: 將會議逐字或筆記整理為：結論、決策、待辦（Owner/DDL）、風險與追蹤問題。
    model: gemini-3-flash-preview
    default_temperature: 0.3
    tags_tc: [會議, 待辦, 決策]
    output_preset: markdown

  - id: email-drafter
    name_en: Professional Email Drafter
    name_tc: 專業郵件撰寫師
    category_tc: 生產力
    description_tc: 依情境撰寫清楚、禮貌、有行動呼籲的郵件；可提供強硬/溫和/中性三版本。
    model: gemini-3-flash-preview
    default_temperature: 0.4
    tags_tc: [郵件, 商務溝通, 三版本]
    output_preset: markdown

  - id: code-reviewer
    name_en: Code Reviewer
    name_tc: 程式碼審查員
    category_tc: 工程
    description_tc: 針對前端程式碼提出可維護性、安全性、效能與可讀性建議；提供具體修改片段與理由。
    model: gemini-3-flash-preview
    default_temperature: 0.25
    tags_tc: [Code Review, 品質, 安全]
    output_preset: markdown

  - id: refactor-coach
    name_en: Refactoring Coach
    name_tc: 重構教練
    category_tc: 工程
    description_tc: 將雜亂程式碼重整為清楚模組，降低耦合；提供分步驟重構路線與回歸驗證清單。
    model: gemini-3-flash-preview
    default_temperature: 0.3
    tags_tc: [重構, 模組化, 回歸]
    output_preset: markdown

  - id: api-design-advisor
    name_en: API Design Advisor
    name_tc: API 設計顧問
    category_tc: 工程
    description_tc: 協助設計前端 service layer 介面、錯誤模型、重試策略、串流輸出介面（未來 roadmap）。
    model: gemini-3-flash-preview
    default_temperature: 0.3
    tags_tc: [API, Service layer, 串流]
    output_preset: markdown

  - id: compliance-checker
    name_en: Compliance Checker
    name_tc: 合規檢核員
    category_tc: 法務/合規
    description_tc: 針對文件或流程做合規風險掃描（個資、授權、保存、告知義務），輸出風險與改善建議（非法律意見）。
    model: gemini-3-flash-preview
    default_temperature: 0.2
    tags_tc: [合規, 個資, 風險]
    output_preset: markdown

  - id: contract-analyst
    name_en: Contract Analyst
    name_tc: 合約條款分析師
    category_tc: 法務/合規
    description_tc: 針對合約條款做摘要、疑點、對我方不利條款、建議修訂文字（需使用者法務複核）。
    model: gemini-3-flash-preview
    default_temperature: 0.25
    tags_tc: [合約, 條款, 修訂建議]
    output_preset: markdown
5) 
