
# Prompt: Build the Artistic Intelligence Workspace v2.0

## 1. Core Objective

Your primary task is to build a sophisticated, browser-based AI studio called "Artistic Intelligence Workspace v2.0". This application serves as a multi-functional platform for interacting with generative AI models, featuring a stunning, dynamically themed user interface.

The application has three main views:
1.  **Dashboard**: An overview of AI usage with key metrics and charts.
2.  **Agent Studio**: A powerful environment to run and manage AI agents, including editing, uploading, downloading, and AI-powered repair of agent configurations.
3.  **Document Intelligence**: A tool to process and analyze various document types (text, PDF, images) to extract summaries and insights.

A key feature is the "Painter Style" theming, which allows the user to dynamically change the entire application's appearance based on the styles of famous artists. The application must also support internationalization for English (en) and Traditional Chinese (tc).

## 2. Technology Stack & Setup

You must use the following technologies. Adhere strictly to this stack.

*   **Framework**: Next.js 15+ (App Router)
*   **Language**: TypeScript
*   **UI Components**: ShadCN UI. You will need to build out a full suite of standard components (`Button`, `Card`, `Input`, `Select`, `Tabs`, etc.).
*   **Styling**: Tailwind CSS. All styling should be done via Tailwind classes and CSS variables defined in `src/app/globals.css`.
*   **Icons**: `lucide-react`.
*   **AI/GenAI**: Genkit (`@genkit-ai/next`, `@genkit-ai/google-genai`) for structured AI flows and the Google Generative AI SDK (`@google/generative-ai`) for direct model calls.
*   **State Management**: React Hooks (`useState`, `useEffect`, `useContext`). State should be managed at the top level (`src/app/page.tsx`) and passed down as props.
*   **YAML**: `js-yaml` for parsing and dumping agent configuration files.
*   **Charts**: `recharts` integrated with ShadCN's chart components.

### `package.json` Dependencies

Ensure the `package.json` file includes the following key dependencies:

```json
"dependencies": {
    "@genkit-ai/google-genai": "^1.20.0",
    "@genkit-ai/next": "^1.20.0",
    "@google/generative-ai": "^0.16.0",
    "@radix-ui/react-accordion": "^1.2.3",
    "class-variance-authority": "^0.7.1",
    "clsx": "^2.1.1",
    "genkit": "^1.20.0",
    "js-yaml": "4.1.0",
    "lucide-react": "^0.475.0",
    "next": "15.5.9",
    "react": "^19.2.1",
    "react-dom": "^19.2.1",
    "recharts": "^2.15.1",
    "tailwind-merge": "^3.0.1",
    "tailwindcss-animate": "^1.0.7",
    "zod": "^3.24.2"
  },
```
*(Note: Include all necessary `@radix-ui` and other ShadCN dependencies as well).*

## 3. Project Structure

Organize the project using the Next.js App Router. The `src` directory should be structured as follows:

```
src/
├── ai/
│   ├── flows/
│   │   ├── repair-invalid-yaml.ts
│   │   └── summarize-document-intelligence.ts
│   ├── genkit.ts
│   └── dev.ts
├── app/
│   ├── actions.ts
│   ├── globals.css
│   ├── layout.tsx
│   └── page.tsx
├── components/
│   ├── ui/                 # All ShadCN UI components
│   ├── agent-studio.tsx
│   ├── app-shell.tsx
│   ├── dashboard.tsx
│   ├── doc-intelligence.tsx
│   └── main-sidebar.tsx
├── hooks/
│   ├── use-mobile.tsx
│   └── use-toast.ts
├── lib/
│   ├── constants.ts
│   ├── types.ts
│   └── utils.ts
public/
└── agents.yaml
```

## 4. Detailed Feature Implementation

### 4.1. Core Layout & State Management (`app/page.tsx`, `components/app-shell.tsx`)

1.  **Main State (`page.tsx`)**: The root `Home` component in `page.tsx` will manage the application's primary state using `React.useState`:
    *   `apiKey: string`: Stores the user's Gemini API key **in memory only**. It must not be persisted in `localStorage` or cookies.
    *   `agents: Agent[]`: The list of available AI agents.
    *   `currentView: ViewType`: The currently displayed view (`dashboard`, `agent-studio`, or `doc-intelligence`).
    *   `settings: AppSettings`: An object containing `theme`, `language`, and `painterStyle`.

2.  **Agent Loading (`page.tsx`)**: In a `useEffect` hook, fetch the initial agent definitions from `/agents.yaml`, parse the YAML using `js-yaml`, and set the `agents` state. Handle errors gracefully with a toast notification and fall back to a default set of agents defined in `constants.ts`.

3.  **App Shell (`app-shell.tsx`)**: This component is the main layout. It receives all state and handler functions from `page.tsx`.
    *   It should use a custom `SidebarProvider` component (based on `shadcn-ui-sidebar-pro`).
    *   It renders `MainSidebar` and the main content area.
    *   It contains a `useEffect` hook to toggle the `light`/`dark` class on the `<html>` element based on `settings.theme`.
    - It contains another `useEffect` to apply the dynamic "Painter Style" CSS variables to the root element based on `settings.painterStyle`.
    *   It uses a `switch` statement to render the correct view component (`Dashboard`, `AgentStudio`, `DocumentIntelligence`) based on the `currentView` prop.

### 4.2. Dynamic Theming & Internationalization (`lib/constants.ts`)

1.  **Painter Styles**:
    *   In `lib/constants.ts`, define an array `PAINTER_STYLES` containing the names of all 20 artists (e.g., `van_gogh`, `dali`).
    *   Define a large object `PAINTER_CSS` that maps each painter style to a dictionary of CSS variables and their values (e.g., `--wow-bg`, `--wow-fg`, `--wow-font-headline`). These will be applied dynamically in `app-shell.tsx`.
    *   In `globals.css`, use these variables in the `body` and create utility classes like `.font-headline` to consume them.
    *   In `tailwind.config.ts`, configure `fontFamily` to use these CSS variables as the primary fonts.

2.  **Internationalization (i18n)**:
    *   In `lib/constants.ts`, create a `LABELS` object. It should have two keys: `en` and `tc`.
    *   Each language key will contain a nested object with all UI strings for the sidebar, dashboard, and other components.
    *   All components must consume labels from this object based on the `language` prop passed down from `page.tsx`.

### 4.3. Main Sidebar (`components/main-sidebar.tsx`)

The sidebar is the control center for the application.

*   **Navigation**: A list of buttons to set the `currentView` state (`Dashboard`, `Agent Studio`, `Doc Intelligence`). Use `lucide-react` icons for each. The active item should be highlighted.
*   **API Key Input**:
    *   An `Input` field for the Gemini API key.
    *   A `Button` with an `Eye` / `EyeOff` icon to toggle the input's type between `text` and `password`.
    *   A `Badge` below the input to show the `Status`: "Operational" if the key exists, "Key Missing" if it's empty. Style the badge accordingly (e.g., green for operational, red for missing).
*   **Appearance Settings**:
    *   **Painter Style**: A `Select` dropdown populated with `PAINTER_STYLES`. It controls the `painterStyle` setting. Include a "Jackpot!" button with a `Dices` icon next to it that, when clicked, selects a random style.
    *   **Theme**: A `Switch` component to toggle between `light` and `dark` mode, controlling the `theme` setting. Display `Sun` or `Moon` icons next to the label.
    *   **Language**: A `Switch` component to toggle between `en` and `tc`, controlling the `language` setting. Display the current language code (EN/TC) next to the switch.

### 4.4. Dashboard View (`components/dashboard.tsx`)

This is a read-only view that displays mock data.

*   **Metrics**: Display three `Card` components for "Total Runs", "Active Agents", and "Avg. Latency". Use `lucide-react` icons in each card header.
*   **Charts**:
    *   Display two `Card` components, each containing a chart.
    *   The first chart is a `LineChart` for "Token Usage Trends".
    *   The second is a `BarChart` for "Model Distribution".
    *   Both charts must use `recharts` and the ShadCN `ChartContainer` wrapper.
    *   All data for metrics and charts must be sourced from `MOCK_METRICS` and `MOCK_CHART_DATA` in `lib/constants.ts`.
*   Style all cards with the `glass-panel` class for a semi-transparent, blurred background effect.

### 4.5. Agent Studio View (`components/agent-studio.tsx`)

This view is for interacting with AI agents and is split into two tabs.

*   **Tabs**: Use ShadCN's `Tabs` component for "Run" and "Manage" modes.
*   **Run Tab**:
    *   A `Select` dropdown to choose an agent from the `agents` prop.
    *   An `Input` field to optionally override the agent's default model.
    *   Two main panels side-by-side:
        1.  **Input**: A `Textarea` for the user's prompt.
        2.  **Output**: A `Card` to display the AI's response. It should show a placeholder text initially, display a `Loader2` spinning icon while loading, and then show the formatted output in a `<pre>` tag. Include a `Copy` button.
    *   A "Run Agent" `Button` at the bottom. This button calls the `generateContentAction` server action. It should be disabled during the API call.

*   **Manage Tab**:
    *   A large `Textarea` that displays the current agent list as a YAML string. It should be editable.
    - The initial value is derived from `yaml.dump({ agents })`.
    *   Three buttons at the bottom:
        1.  **Upload YAML**: Triggers a hidden file input. When a user selects a `.yaml` file, read its content, parse it with `js-yaml`, and update the application's `agents` state via the `onUpdateAgents` callback. If parsing fails, show a toast and display the invalid YAML in the textarea for repair.
        2.  **Download YAML**: Creates a blob from the `yamlText` state and triggers a file download for `agents.yaml`.
        3.  **AI Repair**: A `Button` with a `Wand2` icon that calls the `repairYamlAction` server action, passing the current (potentially invalid) text from the YAML textarea. On success, it updates the `agents` state and the textarea content.

### 4.6. Document Intelligence View (`components/doc-intelligence.tsx`)

This view analyzes documents.

*   **Tabs**: Use `Tabs` for "Paste Text" and "Upload File" modes.
*   **Input Area**:
    *   **Paste Tab**: A `Textarea` for direct text input.
    *   **Upload Tab**: A `Card` that acts as a dropzone. It should show an `Upload` icon and prompt. When a file is selected (txt, md, pdf, png, jpg), it should show a preview (an `Image` component for images, a `FileText` icon for others) along with the file name and size.
*   **Output Area**:
    *   A large `Card` containing an editable `Textarea` for the AI-generated output.
    *   Include a toolbar above the output with a `Slider` to control the `fontSize` of the output text.
    *   Include two `Download` buttons to save the output as `.md` or `.txt`.
*   **Action Button**: A "Process Document" `Button` that calls the `summarizeDocumentAction` server action. It should handle both pasted text and uploaded files, converting them to a `dataURI` format for the action.

### 4.7. Backend and AI Logic

1.  **Server Actions (`app/actions.ts`)**:
    *   `generateContentAction`: This action takes the API key, prompt, model, and other parameters. It instantiates the `GoogleGenerativeAI` class from `@google/generative-ai` and calls `generateContent`. It must have robust error handling, especially for invalid API keys.
    *   `repairYamlAction`: This is a thin wrapper that calls the `repairInvalidYaml` Genkit flow.
    *   `summarizeDocumentAction`: This is a thin wrapper that calls the `summarizeDocumentIntelligence` Genkit flow.

2.  **Genkit Setup (`ai/genkit.ts`)**:
    *   Initialize a global `ai` instance from Genkit.
    *   Configure it with the `googleAI()` plugin.
    *   Set a default model like `gemini-2.5-flash`.

3.  **Genkit Flows (`ai/flows/*.ts`)**:
    *   **`repair-invalid-yaml.ts`**:
        *   Define `RepairInvalidYamlInputSchema` (with `rawText: z.string()`) and `RepairInvalidYamlOutputSchema` (with `repairedYaml: z.string()`).
        *   Create a prompt (`repairAgentYamlPrompt`) instructing the AI to act as a YAML expert and fix the provided text.
        *   Define the `repairInvalidYamlFlow` using `ai.defineFlow`.
        *   Export a wrapper function `repairInvalidYaml` that calls the flow.
    *   **`summarize-document-intelligence.ts`**:
        *   Define `SummarizeDocumentIntelligenceInputSchema` (with `documentDataUri: z.string()`).
        *   Define `SummarizeDocumentIntelligenceOutputSchema` (with `summary: z.string()`).
        *   Create a prompt instructing the AI to summarize the document provided via a `{{media url=documentDataUri}}` Handlebars helper.
        *   Define and export the flow and its wrapper function.

## 5. Final Polish

*   Ensure all components are responsive and work well on mobile devices.
*   Use the `useToast` hook for all user feedback (success, error).
*   Ensure loading states are handled correctly with spinners (`Loader2` icon).
*   The API key must be passed from the root page down to any component or action that needs it. Do not use context or global stores for the API key.
*   Make sure all UI elements are fully translated based on the selected language.

Your final output should be a complete, functional application matching the one described. Good luck!
