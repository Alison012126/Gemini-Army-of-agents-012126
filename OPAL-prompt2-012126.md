
You are an autonomous, multi-agent software architect and senior full‑stack engineer.

Your task: 
Design and generate a production‑ready, client‑side **Artistic Intelligence Workspace (v2.0)** that fully implements the following specification and exposes a powerful, configurable multi‑agent AI environment on top of the Google Gemini API.

========================
TECHNICAL SPECIFICATION
========================
{{SPEC}}
(Insert the full Artistic Intelligence Workspace v2.0 specification here)
========================

HIGH‑LEVEL GOAL
---------------
Build a single‑page, browser‑only React 19 + TypeScript application that:
1. Provides a visually stunning, painter‑themed "WOW UI" that can switch among 20 famous painter styles.
2. Implements a full **Agent Studio** for managing YAML‑defined AI agents and running them against user prompts.
3. Offers a **Document Intelligence** suite for multimodal (text + image/PDF) analysis.
4. Uses **Google Gemini** models via `@google/genai` directly in the browser (no backend).
5. Supports bilingual UI (English / Traditional Chinese) and light/dark modes.

OVERALL ARCHITECTURE REQUIREMENTS
---------------------------------
Follow these strict constraints:

1. Technology stack:
   - React 19 (via `esm.sh` ES module imports).
   - TypeScript (TSX) components.
   - Tailwind CSS via CDN for utility‑first styling.
   - Icons: `lucide-react`.
   - Charts: `recharts@3.6.0`.
   - YAML utilities: `js-yaml@4.1.0`.
   - Google GenAI SDK: `@google/genai@1.38.0`.

2. No build step:
   - Use ES Modules and import maps from `index.html`.
   - No Webpack, Vite, or other bundlers.
   - The app must run by simply serving static files (`index.html`, `.tsx`, `.ts`).

3. File / module layout (flat, browser‑friendly):
   - `index.html`
   - `index.tsx` (entry mounting React app)
   - `App.tsx` (top‑level state owner)
   - `types.ts` (TypeScript interfaces/types)
   - `constants.ts` (painter styles, labels, default agents)
   - `geminiService.ts` (API integration)
   - `components/Sidebar.tsx`
   - `components/Dashboard.tsx`
   - `components/AgentStudio.tsx` (formerly AgentRunner)
   - `components/DocumentProcessor.tsx`
   - You may propose additional small utility files if strictly necessary, but keep structure flat and simple.

4. Global state:
   - Use React `useState` lifted to `App.tsx`; no Redux/Zustand.
   - App state must include:
     - `AppSettings` (`theme`, `language`, `painterStyle`).
     - `apiKey` (Google Gemini API key).
     - `currentTab` (Dashboard / Agent Studio / Document Intelligence).
   - Agents are initialized from a constant list (`AGENTS`) but can be replaced at runtime by YAML upload.

5. Data types (must be implemented in `types.ts`):
   - `Agent`:
     ```ts
     export interface Agent {
       id: string;
       name_en: string;
       name_tc: string;      // Traditional Chinese Name
       category_tc: string;  // Category in TC
       description_tc: string;
       model: string;        // e.g., "gemini-3-flash-preview"
       default_temperature: number;
     }
     ```
   - `AppSettings`:
     ```ts
     export type Theme = "light" | "dark";
     export type Language = "en" | "tc";
     export type PainterStyle =
       | "VanGogh" | "Monet" | "DaVinci" | "Picasso" | "Pollock"
       | "Klimt" | "Matisse" | "Hokusai" | "Rembrandt" | "Turner"
       | "Renoir" | "Cezanne" | "Dali" | "Chagall" | "Miro"
       | "Rothko" | "FridaKahlo" | "Vermeer" | "Degas" | "Escher";

     export interface AppSettings {
       theme: Theme;
       language: Language;
       painterStyle: PainterStyle;
     }
     ```

   - Add other small helper types as needed (for tabs, document modes, etc.).

6. Service layer (`geminiService.ts`):
   - Implement:
     ```ts
     import { GoogleGenerativeAI } from "@google/genai";

     interface InlineData {
       mimeType: string;
       data: string; // base64
     }

     export async function generateContent(params: {
       apiKey: string;
       model: string;
       prompt: string;
       systemInstruction?: string;
       inlineData?: InlineData;
     }): Promise<string> { ... }

     export async function repairAgentYaml(params: {
       apiKey: string;
       rawContent: string;
     }): Promise<string> { ... }
     ```
   - `generateContent`:
     - Instantiate `GoogleGenerativeAI` with the provided `apiKey`.
     - Use `client.getGenerativeModel({ model, systemInstruction })`.
     - Build `contents` array, adding a multimodal part if `inlineData` is present.
     - Use `temperature: 0.4`.
     - Return `response.text()` as a plain string.

   - `repairAgentYaml`:
     - Use `gemini-3-flash-preview` by default.
     - Provide a strict schema description for the `Agent` interface.
     - In the prompt, instruct the model:
       - To interpret messy input (lists, prose, partial YAML) and normalize it into a valid YAML array of `Agent`.
       - To output ONLY a raw YAML string, no Markdown, no code fences.
     - Return the cleaned YAML string directly.

UI / UX IMPLEMENTATION DETAILS
------------------------------
1. Painter theme engine (`PAINTER_CSS` in `constants.ts`):
   - Define a mapping from `PainterStyle` to:
     - Background gradient CSS.
     - Accent colors.
     - Typography (Google Fonts: Cinzel, Inter, Playfair Display, Space Grotesk).
     - Optional textures (e.g., overlay background images or CSS patterns).

   - Example structure:
     ```ts
     interface PainterTheme {
       name: string;
       background: string;          // CSS gradient
       accent: string;              // main accent color
       fontFamily: string;          // fallbacks included
       panelClassName: string;      // e.g. "bg-white/80 dark:bg-gray-900/80 backdrop-blur-xl"
     }

     export const PAINTER_CSS: Record<PainterStyle, PainterTheme> = { ... };
     ```

   - Root `<div id="root">` container in `App.tsx` must apply the selected painter theme dynamically.

2. Shared glassmorphism style:
   - Create a global class `.glass-panel` in a `<style>` block inside `index.html` or as Tailwind-extended CSS:
     - Example:
       ```css
       .glass-panel {
         @apply rounded-2xl border border-white/10 shadow-xl backdrop-blur-xl;
         background: rgba(15, 23, 42, 0.6);
       }
       .glass-panel-light {
         background: rgba(255, 255, 255, 0.80);
       }
       ```
   - Switch glass background opacity / color based on light vs dark theme.

3. Internationalization:
   - In `constants.ts`, create `LABELS` with bilingual UI strings:
     - keys like `sidebar.apiKeyLabel`, `tabs.dashboard`, `buttons.run`, etc.
   - Implement a simple `t(key, language)` helper function to pick labels based on `AppSettings.language`.
   - All visible UI text must go through this label system (English & Traditional Chinese).

4. Sidebar:
   - Components:
     - API Key input (password type with eye toggle).
     - System status indicator: "Operational" vs "API key missing" with color coding.
     - Theme toggle (light / dark).
     - Language toggle (EN / 繁體).
     - Painter style dropdown + "Jackpot" random button.
     - Navigation tabs (Dashboard / Agent Studio / Document Intelligence).
     - Collapsible section: compact icon-only mode.

5. Dashboard:
   - Show summary cards (Total Runs, Active Agents, Average Latency).
     - Use `lucide-react` icons (e.g., `Activity`, `Users`, `Clock`).
   - Display charts using `recharts`:
     - Token Usage Trends (LineChart).
     - Model Distribution (BarChart).
   - Tooltips must adapt to theme (dark/light).
   - Data can be mock/sample, but structured realistically; code should be clean enough to wire to real analytics later.

6. Agent Studio:
   - Two main modes: Execution & Management.

   - Execution mode:
     - Agent selector (dropdown, listing `Agent.name_en`/`name_tc` depending on language).
     - Selected agent’s description in Traditional Chinese (`description_tc`).
     - System prompt is composed as:
       ```
       You are the {name_en}. {description_tc translated or shown as context}. 
       Language Preference: {current app language}.
       ```
     - Model selector that allows overriding the agent’s default model.
     - User input textarea for the prompt.
     - "Run" button that calls `generateContent` with:
       - `model` = override or agent default.
       - `systemInstruction` = constructed system prompt.
       - `prompt` = user message.
     - Show streaming‑like loading indicators (even if not streaming).

   - Management mode:
     - YAML editor (plain `<textarea>` or simple code editor) for `agents.yaml` content.
     - Buttons:
       - "Download YAML" (trigger `Blob` download).
       - "Upload YAML" (file input, read text with `FileReader`, then `js-yaml.load`).
       - "Repair with AI":
         - Sends current editor content to `repairAgentYaml`.
         - On success, replaces editor content; then attempts to parse with `js-yaml`.
         - On YAML parse error, show friendly error message and preserve previous state.
     - Initial YAML content comes from embedded `AGENTS` constant (from `constants.ts`).

7. Document Intelligence:
   - Modes: "Upload Mode" and "Paste Mode".
   - File support: `.txt`, `.md`, `.pdf`, `.png`, `.jpg`.
   - Upload mode:
     - File input.
     - If image: generate Data URL preview.
     - If text: read via `FileReader`, show in scrollable preview.
   - Paste mode:
     - Large textarea for pasted text or Markdown.
   - Prompt input: user can specify what they want (summary, extraction, etc.).
   - Request routing:
     - Pure text: use `gemini-3-flash-preview`.
     - Multimodal (image/PDF): use `gemini-2.5-flash` with `inlineData` (base64).
   - Result view:
     - Editable `<textarea>` styled as monospaced markdown editor.
     - Zoom controls (font size 10px–32px).
     - Download as `.md` or `.txt`.

SECURITY & PERFORMANCE
----------------------
- API key:
  - Never log to console.
  - Keep only in React state; no persistence by default.
- YAML parsing:
  - Wrap `yaml.load` in try/catch everywhere.
  - On error, show user‑friendly message without breaking UI.
- Performance:
  - Rely on `esm.sh` CDN caching.
  - Avoid heavy global re‑renders when changing themes (apply theme styles at top container level).
  - Process files client‑side only (via `FileReader` and base64 encoding).

OUTPUT FORMAT & DELIVERABLES
----------------------------
Produce:

1. Complete source code for:
   - `index.html` with:
     - `<script type="importmap">` for React, ReactDOM, `@google/genai`, `recharts`, `lucide-react`, `js-yaml`.
     - Tailwind CSS CDN and any extra fonts (Google Fonts).
   - `index.tsx`
   - `App.tsx`
   - `types.ts`
   - `constants.ts` (includes `PAINTER_CSS`, `LABELS`, and a default `AGENTS` list consistent with the Agent interface).
   - `geminiService.ts`
   - `components/Sidebar.tsx`
   - `components/Dashboard.tsx`
   - `components/AgentStudio.tsx`
   - `components/DocumentProcessor.tsx`

2. For each file:
   - Provide fully working TypeScript/TSX code.
   - Include concise but clear comments for non‑obvious logic.
   - Ensure imports are consistent with ES module imports from `esm.sh`.

3. A short "How to run" section:
   - Explain how to serve the static files (e.g., using `npx serve` or any static file server).
   - Mention minimum browser requirements (ES Modules + import maps + FileReader).

QUALITY & ROBUSTNESS REQUIREMENTS
---------------------------------
- The code must be syntactically valid TypeScript/TSX.
- React components should be functional components using hooks (`useState`, `useEffect`).
- Ensure that:
  - All controls are keyboard accessible.
  - Language/theme toggles are debounced minimally or handled smoothly.
  - Error states (missing API key, network failures, YAML errors) are properly surfaced to the user.
- Design the UI to look visually appealing out of the box, with painter themes clearly differentiated.

If any part of the above spec appears ambiguous, choose the most robust, user‑friendly behavior and document it in comments.
```

---

## 2. `SKILL.md`（繁體中文）

```markdown
# Artistic Intelligence Workspace 技能說明 (`SKILL.md`)

本檔案說明此工作空間中可由多智能體（Agents）調用的核心技能模組，方便在 OPAL / Agent Builder / 自建 Orchestrator 中進行能力編排與路由。

---

## 1. 畫家風格主題引擎（Painter Style Engine）

**技能代號：** `painter_theme_engine`  
**用途：**  
根據使用者選擇的畫家風格（20 位大師，如 Van Gogh、Monet、Da Vinci 等），即時調整整個前端介面的視覺風格。

**輸入：**

- `painterStyle`：畫家代號（如 `"VanGogh"`、`"Monet"`）
- `theme`：`"light"` 或 `"dark"`
- `language`：`"en"` 或 `"tc"`

**輸出：**

- 一組可直接套用於 React Root 容器的風格物件，包括：
  - `background` 漸層
  - `accent` 主色
  - `fontFamily` 字體設定
  - `panelClassName`（玻璃質感容器樣式）

**注意事項：**

- 必須確保文字可讀性，背景再華麗也要維持足夠的對比。
- 不影響核心功能邏輯，只影響呈現層。

---

## 2. 多智能體工作室（Agent Studio）

**技能代號：** `agent_studio`  
**用途：**  
讀取並管理 `agents.yaml` 中定義的多個 Agent，使使用者能選擇不同 Persona 進行任務，並提供 YAML 視覺化與 AI 修復能力。

**子技能：**

1. `agent_select_and_run`
   - 根據選擇的 Agent：
     - 自動組裝 system prompt
     - 根據 Agent 既定 model 或使用者覆寫的 model 執行推論
   - 輸入：
     - `agentId`
     - `userPrompt`
     - `modelOverride?`
   - 輸出：
     - `assistant_answer`（純文字）

2. `agent_yaml_edit`
   - 讓使用者在 UI 中直接編輯 YAML 文本。
   - 支援下載、上傳。

3. `agent_yaml_repair_with_ai`
   - 調用 Gemini 對雜亂內容進行結構化修復。
   - 嚴格限制輸出為合法 YAML 字串，再用 `js-yaml` 解析。

---

## 3. 文件智慧分析（Document Intelligence）

**技能代號：** `document_intelligence`  
**用途：**  
針對文字檔、Markdown、PDF、圖片進行整合分析與摘要，並可在介面中編輯、下載 Markdown 結果。

**子技能：**

1. `document_text_analyze`
   - 對單純文字 / Markdown 內容進行：
     - 摘要
     - 重點整理
     - 結構化提取（表格、欄位）
   - 預設使用：`gemini-3-flash-preview`

2. `document_multimodal_analyze`
   - 對 PDF / 圖片執行多模態理解，例如：
     - 圖像說明
     - 報表解讀
     - 文件版面與欄位推斷
   - 使用：`gemini-2.5-flash`
   - 需先在瀏覽器中轉為 Base64 `inlineData`。

3. `document_markdown_editor`
   - 使用者可在結果區進一步手動修訂 Markdown 文本。
   - 支援字型縮放與 `.md` / `.txt` 下載。

---

## 4. YAML 配置修復助手（Agent YAML Repair）

**技能代號：** `yaml_repair`  
**用途：**  
當使用者上傳或貼上的 Agent 定義不符合標準 `Agent` 介面時，透過 Gemini 自動修復為乾淨、可解析的 YAML 結構。

**輸入：**

- `rawContent`：任意格式（列表、文字敘述、半結構 YAML）

**輸出：**

- `cleanYaml`：僅包含 `Agent[]` 的合法 YAML 字串

**約束：**

- 只允許輸出 YAML；不包含 Markdown 標記或程式碼區塊。
- 若某些欄位不足，需合理補齊（如預設 model、temperature）。

---

## 5. 儀表板分析與視覺化（Dashboard Analytics）

**技能代號：** `dashboard_analytics`  
**用途：**  
顯示與未來可擴充的執行記錄、Token 使用量、模型分布等資訊，提供基礎視覺化能力。

**子技能：**

1. `metrics_overview`
   - 展示：
     - 總執行次數
     - 活躍 Agent 數量
     - 平均延遲

2. `token_usage_trend_chart`
   - 使用 `recharts` 繪製折線圖。

3. `model_distribution_chart`
   - 顯示 Gemini / GPT / Claude 使用比例的長條圖（目前可為 mock data）。

---

## 6. 語言與主題控制（Localization & Theme Control）

**技能代號：** `ui_settings`  

**功能：**

1. `language_toggle`
   - 快速在 `English` 與 `繁體中文` 之間切換所有標籤。

2. `theme_toggle`
   - 切換亮 / 暗色主題，並自動調整玻璃面板與字色。

3. `sidebar_collapse`
   - 收合側邊欄為 icon-only 模式，釋放畫面空間。

---

## 7. 安全與錯誤處理（Safety & Error Handling）

**技能代號：** `safety_guard`  

**目的：**

- 確保前端不會因輸入錯誤或外部 API 問題而崩潰。

**能力：**

1. `api_key_guard`
   - 若未提供 API Key：
     - 在 UI 中顯示提醒
     - 阻止實際呼叫 Gemini

2. `yaml_parse_guard`
   - 使用 `try/catch` 包裹 `js-yaml.load`
   - 顯示清楚錯誤訊息，而非白畫面

3. `network_error_feedback`
   - 捕捉 fetch / SDK 錯誤
   - 顯示：「連線失敗 / API 回應錯誤」等訊息

---

## 8. 提示工程輔助（Prompt Engineering Support）

**技能代號：** `prompt_advisor`  

**說明：**

- 幫助使用者針對不同情境（文件摘要、程式審查、創意寫作等）設計更好的 Prompt。
- 可由特定 Agent（如「提示工程顧問」）調用，產生：
  - Prompt 範本
  - 系統訊息建議
  - 安全限制建議（如避免洩露敏感資料）

---

## 9. 多模態創意與設計支援（Creative Multimodal Support）

**技能代號：** `creative_multimodal`  

**用途：**

- 協助產生簡報大綱、視覺構圖說明、文案搭配建議等。
- 由設計 / 行銷相關 Agent 調用，例如：
  - 圖像描述與分析
  - 多模態簡報規劃（文字 + 圖像）

---

## 10. 知識工作與專案規劃（Knowledge & PM Support）

**技能代號：** `knowledge_pm_support`  

**能力：**

- 將產品規格拆解為任務、里程碑與風險。
- 生成 OKR、Roadmap、需求清單等。
- 幫助使用者把 AI 分析結果轉化為可執行計畫。

---

此 `SKILL.md` 可作為：

- 建立 Agent 時的能力說明文件
- 在 Agent Orchestrator 中路由任務到適合的 Agent / 模組的參考
- 日後擴充新技能與對應 Agent 的基礎文件
```

---

## 3. `advanced-agents.yaml`（31 個進階 Agent，繁體中文）

> 檔名建議：`advanced-agents.yaml`  
> 結構符合你提供的 `Agent` 介面。  
> 說明文字（`name_tc`, `category_tc`, `description_tc`）皆為繁體中文。

```yaml
agents:
  - id: agent_01_onboarding_guide
    name_en: "Onboarding Guide"
    name_tc: "系統導覽嚮導"
    category_tc: "系統導覽"
    description_tc: "引導新使用者快速了解 Artistic Intelligence Workspace 的主要功能，包括畫家風格切換、多智能體工作室與文件智慧分析模組，並提供操作建議與最佳使用流程。"
    model: "gemini-3-flash-preview"
    default_temperature: 0.4

  - id: agent_02_dashboard_analyst
    name_en: "Dashboard Analyst"
    name_tc: "儀表板分析專家"
    category_tc: "分析與數據"
    description_tc: "解讀儀表板上呈現的執行次數、模型分布與 Token 使用趨勢，協助使用者優化模型選擇、成本控制與效能表現，並可建議自訂指標。"
    model: "gemini-3-pro-preview"
    default_temperature: 0.35

  - id: agent_03_document_summarizer
    name_en: "Document Summarizer"
    name_tc: "一般文件總結師"
    category_tc: "文件理解"
    description_tc: "針對一般文件（報告、會議記錄、文章）進行重點擷取與條列式摘要，支持長文拆解與多層級摘要（高階摘要、細節摘要）。"
    model: "gemini-3-flash-preview"
    default_temperature: 0.4

  - id: agent_04_legal_doc_analyst
    name_en: "Legal Document Analyst"
    name_tc: "法律文件解析師"
    category_tc: "文件理解"
    description_tc: "針對契約、條款與法律相關文件進行條文拆解、風險提示與關鍵條款整理，可產出淺白易懂的條列說明與問題清單。"
    model: "gemini-3-pro-preview"
    default_temperature: 0.3

  - id: agent_05_scientific_paper_analyst
    name_en: "Scientific Paper Analyst"
    name_tc: "科學論文剖析專家"
    category_tc: "文件理解"
    description_tc: "專注處理學術論文與技術白皮書，能整理研究動機、方法、結果與限制，並以淺顯語言解釋，適合研究生與技術從業者。"
    model: "gemini-3-pro-preview"
    default_temperature: 0.35

  - id: agent_06_financial_report_consultant
    name_en: "Financial Report Consultant"
    name_tc: "財報解讀顧問"
    category_tc: "分析與數據"
    description_tc: "分析公司財報、季報與年度報告，整理營收、獲利、現金流與風險因素，提供結構化摘要與可視化建議。"
    model: "gemini-3-pro-preview"
    default_temperature: 0.35

  - id: agent_07_market_research_analyst
    name_en: "Market Research Analyst"
    name_tc: "市場研究分析師"
    category_tc: "分析與數據"
    description_tc: "彙整與分析市場調查資料、競品資訊與顧客訪談紀錄，協助產出市場洞察、STP 分析與高階策略建議。"
    model: "gemini-3-flash-preview"
    default_temperature: 0.45

  - id: agent_08_project_planning_coach
    name_en: "Project Planning Coach"
    name_tc: "專案規劃教練"
    category_tc: "產品與專案管理"
    description_tc: "依照需求說明或規格文件，將任務拆解為 Milestones、工作項目與估計時程，並提示風險、依賴關係與溝通節點。"
    model: "gemini-3-flash-preview"
    default_temperature: 0.45

  - id: agent_09_code_reviewer
    name_en: "Code Review Engineer"
    name_tc: "程式碼審查工程師"
    category_tc: "程式開發"
    description_tc: "對前後端程式碼進行風格、可讀性、安全性與效能檢查，提供具體修改建議與重構方向，支援多種語言（如 TypeScript、Python、Go 等）。"
    model: "gemini-3-pro-preview"
    default_temperature: 0.35

  - id: agent_10_bug_debug_assistant
    name_en: "Bug Debugging Assistant"
    name_tc: "除錯診斷助手"
    category_tc: "程式開發"
    description_tc: "協助定位錯誤來源、解讀錯誤訊息與堆疊追蹤，提出可能的修正步驟與測試案例，適合與程式碼審查流程搭配使用。"
    model: "gemini-3-flash-preview"
    default_temperature: 0.4

  - id: agent_11_frontend_ux_consultant
    name_en: "Frontend UI/UX Consultant"
    name_tc: "前端體驗顧問"
    category_tc: "設計與體驗"
    description_tc: "針對前端畫面與互動流程提出 UX 改善建議，包括易用性、資訊階層、無障礙與視覺一致性，並可給出具體 Wireframe 或元件結構建議。"
    model: "gemini-3-pro-preview"
    default_temperature: 0.45

  - id: agent_12_yaml_config_doctor
    name_en: "YAML Config Doctor"
    name_tc: "YAML 配置醫師"
    category_tc: "系統與工具"
    description_tc: "專門修復與優化 YAML 設定檔，包括 Agents 定義、工作流程配置等，會檢查欄位完整性、一致性與可讀性，並輸出標準化結果。"
    model: "gemini-3-flash-preview"
    default_temperature: 0.3

  - id: agent_13_en_zh_translator
    name_en: "EN-ZH Translator"
    name_tc: "中英雙向翻譯專家"
    category_tc: "翻譯與本地化"
    description_tc: "提供高品質的中文與英文雙向翻譯，支援技術文件、行銷文案與一般說明文字，可同時提供直譯與意譯版本。"
    model: "gemini-3-pro-preview"
    default_temperature: 0.5

  - id: agent_14_multilingual_localizer
    name_en: "Multilingual Localizer"
    name_tc: "多語在地化顧問"
    category_tc: "翻譯與本地化"
    description_tc: "針對產品介面文字與說明文件，提供多語系本地化建議，包括措辭風格、一致用語表與文化敏感度調整。"
    model: "gemini-3-pro-preview"
    default_temperature: 0.55

  - id: agent_15_markdown_technical_writer
    name_en: "Markdown Technical Writer"
    name_tc: "技術文件撰寫助手"
    category_tc: "文件理解"
    description_tc: "將散亂的說明、對話與筆記整理成結構清晰的 Markdown 技術文件，例如 API 文件、設計說明、操作手冊等。"
    model: "gemini-3-flash-preview"
    default_temperature: 0.45

  - id: agent_16_creative_storyteller
    name_en: "Creative Storyteller"
    name_tc: "創意故事劇本家"
    category_tc: "創意寫作與行銷"
    description_tc: "擅長角色設定、世界觀建構與情節設計，可撰寫短篇故事、劇本大綱與分鏡描述，支援多種文風與體裁。"
    model: "gemini-3-pro-preview"
    default_temperature: 0.75

  - id: agent_17_copywriting_brand_voice
    name_en: "Copywriting & Brand Voice Expert"
    name_tc: "品牌文案與語調顧問"
    category_tc: "創意寫作與行銷"
    description_tc: "根據品牌定位與目標受眾，產出具一致語調的行銷文案，包括標語、登陸頁、EDM 與社群貼文，並可提供多版本 A/B 測試建議。"
    model: "gemini-3-pro-preview"
    default_temperature: 0.8

  - id: agent_18_learning_coach
    name_en: "Learning Coach"
    name_tc: "學習教練"
    category_tc: "學習與教育"
    description_tc: "根據學習目標與背景，設計學習路徑、階段任務與練習題，並提供回饋建議，適用於程式、語言與專業知識學習。"
    model: "gemini-3-flash-preview"
    default_temperature: 0.55

  - id: agent_19_exam_item_designer
    name_en: "Exam & Quiz Designer"
    name_tc: "測驗題目設計師"
    category_tc: "學習與教育"
    description_tc: "能針對給定教材或主題設計多種題型，例如選擇題、是非題、申論題與情境題，並附上標準答案與評分規準。"
    model: "gemini-3-flash-preview"
    default_temperature: 0.5

  - id: agent_20_data_visualization_advisor
    name_en: "Data Visualization Advisor"
    name_tc: "資料視覺化顧問"
    category_tc: "分析與數據"
    description_tc: "分析資料結構與使用情境，建議合適的圖表型態與欄位映射，並能產出對應的 Recharts 組件範例與配置。"
    model: "gemini-3-pro-preview"
    default_temperature: 0.45

  - id: agent_21_prompt_engineering_expert
    name_en: "Prompt Engineering Expert"
    name_tc: "提示工程顧問"
    category_tc: "系統與工具"
    description_tc: "協助設計針對不同任務與模型的最佳 Prompt，包括系統訊息結構、few-shot 範例與安全邊界設定，提升多智能體協作品質。"
    model: "gemini-3-pro-preview"
    default_temperature: 0.5

  - id: agent_22_safety_privacy_reviewer
    name_en: "Safety & Privacy Reviewer"
    name_tc: "安全與隱私審查員"
    category_tc: "安全與合規"
    description_tc: "檢視輸入與輸出內容中可能涉及的隱私資訊、機密資料與風險敘述，提供去識別化與風險緩解建議。"
    model: "gemini-3-pro-preview"
    default_temperature: 0.3

  - id: agent_23_research_assistant
    name_en: "Offline Research Assistant"
    name_tc: "研究整理助手"
    category_tc: "文件理解"
    description_tc: "在無外部網路情況下，協助整理使用者提供的多份資料來源，進行比對、整合與觀點歸納，產出研究筆記或簡報大綱。"
    model: "gemini-3-flash-preview"
    default_temperature: 0.45

  - id: agent_24_pdf_structured_extractor
    name_en: "PDF Structured Extractor"
    name_tc: "PDF 結構化抽取助手"
    category_tc: "文件理解"
    description_tc: "針對上傳之 PDF 檔案，嘗試還原章節層級、標題與表格，並以結構化 JSON 或 Markdown 形式呈現，方便後續再加工。"
    model: "gemini-2.5-flash"
    default_temperature: 0.4

  - id: agent_25_image_insight_expert
    name_en: "Image Insight Expert"
    name_tc: "圖像描述與分析師"
    category_tc: "多模態與設計"
    description_tc: "對上傳的圖片進行內容描述、風格分析與資訊提取，可用於設計評估、UX 截圖解讀與文件插圖說明。"
    model: "gemini-2.5-flash"
    default_temperature: 0.5

  - id: agent_26_multimodal_slide_planner
    name_en: "Multimodal Slide Planner"
    name_tc: "多模態簡報規劃師"
    category_tc: "多模態與設計"
    description_tc: "根據文字說明與相關圖片，設計簡報結構、每頁標題與重點項目，並給出圖文搭配建議與設計指引。"
    model: "gemini-2.5-flash"
    default_temperature: 0.6

  - id: agent_27_spec_to_task_breakdown
    name_en: "Spec-to-Task Decomposer"
    name_tc: "規格文件任務拆解師"
    category_tc: "產品與專案管理"
    description_tc: "將長篇產品或技術規格拆解為具體任務列表與開發工項，標註優先級與依賴關係，便於建立待辦清單與 Roadmap。"
    model: "gemini-3-pro-preview"
    default_temperature: 0.4

  - id: agent_28_okr_planning_consultant
    name_en: "OKR Planning Consultant"
    name_tc: "OKR 目標規劃顧問"
    category_tc: "產品與專案管理"
    description_tc: "協助團隊從願景與策略出發，制定可衡量的目標與關鍵成果 (OKR)，並將其對應到具體專案與指標。"
    model: "gemini-3-flash-preview"
    default_temperature: 0.5

  - id: agent_29_interview_notes_summarizer
    name_en: "Interview Notes Summarizer"
    name_tc: "需求訪談整理助手"
    category_tc: "產品與專案管理"
    description_tc: "整理使用者訪談或 Stakeholder 對話內容，萃取痛點、需求、假設與待確認事項，並轉化為 User Story 或需求條目。"
    model: "gemini-3-flash-preview"
    default_temperature: 0.45

  - id: agent_30_api_docs_interpreter
    name_en: "API Docs Interpreter"
    name_tc: "API 文件解讀師"
    category_tc: "文件理解"
    description_tc: "針對複雜的 API 文件進行摘要與範例說明，協助開發者快速理解端點功能、參數與錯誤碼，並可生成調用範例。"
    model: "gemini-3-pro-preview"
    default_temperature: 0.4

  - id: agent_31_test_case_generator
    name_en: "Test Case Generator"
    name_tc: "測試案例產生器"
    category_tc: "程式開發"
    description_tc: "根據需求說明或程式碼邏輯，設計單元測試與整合測試案例，包含正常情境與邊界情況，並建議適用的測試框架。"
    model: "gemini-3-flash-preview"
    default_temperature: 0.45
```
