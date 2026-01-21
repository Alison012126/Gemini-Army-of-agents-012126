
# Prompt for Building the "Artistic Intelligence Workspace"

## 1. Project Overview

You are to create a highly stylized, feature-rich Next.js application called "Artistic Intelligence Workspace". This app serves as a powerful and immersive environment for users to interact with Generative AI models. The key concept is a dynamic UI where the user can select from 20 different "Painter Styles" (e.g., Van Gogh, Monet, Banksy), which dramatically alters the application's entire theme, including background gradients, fonts, and colors, creating a unique "glass panel" aesthetic.

The application will be built using the **Next.js App Router, TypeScript, Tailwind CSS, and ShadCN UI components**. All AI functionality will be powered by **Genkit flows**.

The workspace is divided into three main sections accessible via tabs:
1.  **Dashboard**: Presents mock analytics and usage charts.
2.  **Agent Studio**: Allows users to run and manage a collection of predefined AI "agents" from a YAML configuration file.
3.  **Document Intelligence**: An interface for analyzing pasted text or uploaded documents (including images and PDFs) using multimodal AI models.

A central `WorkspaceProvider` will manage global state such as the current theme, selected language (English and Traditional Chinese), the user's Gemini API key (stored in-session only), and the agent configurations.

## 2. Tech Stack

*   **Framework**: Next.js 15+ (App Router)
*   **Language**: TypeScript
*   **Styling**: Tailwind CSS
*   **UI Components**: ShadCN/UI
*   **AI**: Genkit with the Google AI plugin (`@genkit-ai/google-genai`).
*   **Icons**: `lucide-react`
*   **YAML Parsing**: `js-yaml`
*   **Charts**: `recharts`

## 3. Step-by-Step Implementation Guide

### Milestone 1: Project Setup and Core Layout

1.  **Initialize Project**: Set up a new Next.js project with TypeScript, Tailwind CSS, and ShadCN.
2.  **Global Styles**: In `src/app/globals.css`, define the base HSL color variables for both light and dark themes as provided by ShadCN. Add a `.glass-panel` utility class:
    ```css
    .glass-panel {
        @apply bg-card/60 backdrop-blur-lg border border-white/10 rounded-xl shadow-lg;
    }
    ```
3.  **Font Setup**: In `src/app/layout.tsx`, import and link the following Google Fonts required for the various painter styles: `Inter`, `Space Grotesk`, `Cinzel`, and `Playfair Display`.
4.  **Main Layout (`src/app/page.tsx` & `src/components/artistic-workspace.tsx`)**:
    *   The root page (`page.tsx`) should render a single client component, `<ArtisticWorkspace />`.
    *   Create `artistic-workspace.tsx`. This component will be the core of the UI.
    *   It should contain a flex container with two main children: `<AppSidebar />` and a `<main>` content area.
    *   The `<main>` area will use ShadCN's `<Tabs>` component to create the three primary sections: "Dashboard", "Agent Studio", and "Document Intelligence". Use `LayoutDashboard`, `Bot`, and `ScanText` icons for the tabs.
    *   The tabs list should have the `.glass-panel` style.

### Milestone 2: The "Painter" Theming Engine & Workspace Context

1.  **Types (`src/lib/types.ts`)**: Define the core interfaces for the application: `Agent`, `AppSettings`, `AppLanguage`, `AppTheme`, `PainterStyleName`, and `PainterStyle`.
2.  **Constants (`src/lib/constants.ts`)**:
    *   Create a `PAINTER_STYLES` object that holds the configuration for all 20 painter styles. Each style must have a `name`, `font`, `gradient`, `foreground`, and an optional `overlay` URL.
    *   Create a `LABELS` object for internationalization (i18n). It should contain keys for all UI labels with nested `en` and `tc` translations.
    *   Create mock data (`MOCK_CHART_DATA`) and chart configurations (`TOKEN_USAGE_CHART_CONFIG`, `MODEL_DIST_CHART_CONFIG`) for the dashboard.
3.  **Workspace Context (`src/hooks/use-workspace.ts`)**: Create a React context `WorkspaceContext` to provide shared state across the application. The context type `WorkspaceContextType` should include:
    *   `settings: AppSettings`
    *   `setSettings: (settings: Partial<AppSettings>) => void`
    *   `apiKey: string | null`
    *   `setApiKey: (key: string | null) => void`
    *   `apiStatus: 'missing' | 'operational'`
    *   `agents: Agent[]`
    *   `setAgents: (agents: Agent[]) => void`
    *   `yamlContent: string`
    *   `setYamlContent: (content: string) => void`
    *   `getLabel: (key: string) => string`
4.  **Workspace Provider (`src/components/providers/workspace-provider.tsx`)**:
    *   Implement the `WorkspaceProvider`. It will manage the state for `settings`, `apiKey`, `agents`, and `yamlContent`.
    *   On mount (`useEffect`), it should:
        *   Load `agents.yaml` from the `/public` directory, parse it using `js-yaml`, and populate the `agents` and `yamlContent` state.
        *   Load user settings from `localStorage` and apply them.
    *   It should also contain an effect to apply the `dark` or `light` class to the `<html>` element and persist settings to `localStorage` whenever they change.
    *   The `getLabel` function should return the correct string from the `LABELS` constant based on the current language in settings.
5.  **Theming Logic (`artistic-workspace.tsx`)**:
    *   In `<ArtisticWorkspace />`, use the `useWorkspace` hook to get the current `settings`.
    *   Create a `useEffect` hook that listens for changes to the `settings.painterStyle`.
    *   Inside the effect, update the `document.body.style` for `background` and `color`. Also, manage the font classes on `document.body` based on the selected style's `font` property.
    *   Add a `div` for the `overlay` if the style defines one. This `div` should be positioned absolutely to cover the entire screen with a low opacity.

### Milestone 3: Sidebar and Settings

1.  **Agents YAML (`public/agents.yaml`)**: Create a default `agents.yaml` file in the `public` directory with a few sample agents. Each agent should have an `id`, `name_en`, `name_tc`, `description_tc`, `model`, etc.
2.  **App Sidebar (`src/components/layout/app-sidebar.tsx`)**:
    *   Use the custom ShadCN `Sidebar` component provided in the prompt context. It should be collapsible.
    *   Display the app title and an "Agents" group with a `<Badge>` showing the total number of agents from the `useWorkspace` context.
    *   At the bottom, add a "Settings" button inside a `<Popover>`.
3.  **Settings Popover**:
    *   The popover content should be a `.glass-panel`.
    *   **API Key**: An `<Input type="password">` for the Gemini API Key. Include a button with an `<Eye>`/`<EyeOff>` icon to toggle visibility. A "Save Key" button updates the `apiKey` in the `WorkspaceContext`. Display a badge indicating if the key is "Operational" or "Missing".
    *   **Painter Style**: A `<Select>` dropdown populated with all styles from `PAINTER_STYLES`. Include a `<Button>` with a `<Dices>` icon for a "Jackpot" feature that selects a random style.
    *   **Theme**: A `<Switch>` to toggle between 'light' and 'dark' themes.
    *   **Language**: A button group to toggle between 'en' and 'tc' languages.

### Milestone 4: AI Integration with Genkit

1.  **Genkit Setup (`src/ai/genkit.ts`)**: Configure a global `ai` instance from Genkit, enabling the `googleAI` plugin. Set a default model like `gemini-2.5-flash`.
2.  **Genkit Dev Entry (`src/ai/dev.ts`)**: Create the entry file for the Genkit development server and import all flow files into it.
3.  **Repair YAML Flow (`src/ai/flows/repair-agent-yaml.ts`)**:
    *   Create a Genkit flow `repairAgentYamlFlow`.
    *   **Input Schema**: `z.object({ rawText: z.string() })`
    *   **Output Schema**: `z.object({ repairedYaml: z.string() })`
    *   **Prompt**: Instruct the model to act as a YAML expert, repair the input `rawText`, and return only the valid, repaired YAML content.
    *   Export a wrapper function `repairAgentYaml` that calls the flow.
4.  **Process Document Flow (`src/ai/flows/process-document-intelligence.ts`)**:
    *   Create a flow `processDocumentIntelligenceFlow`.
    *   **Input Schema**: `z.object({ inputType: z.enum(['upload', 'paste']), contentType: z.string().optional(), content: z.string() })`. The `content` will be plain text or a Base64 data URI.
    *   **Output Schema**: `z.object({ processedText: z.string() })`
    *   **Prompt**: Instruct the model to analyze the provided document and return a concise summary or key insights in markdown. Use Handlebars to conditionally process the content as text (`{{{content}}}`) or as media (`{{{media url=content}}}`).
    *   **Logic**: The flow should dynamically select the model: use a multimodal model (e.g., `gemini-2.5-flash`) for images/PDFs and a text model for plain text.
    *   Export a wrapper function `processDocumentIntelligence`.
5.  **Run Agent Flow (`src/ai/flows/run-agent-flow.ts`)**:
    *   Create a new flow `runAgentFlow`.
    *   **Input Schema**: `z.object({ systemInstruction: z.string(), prompt: z.string(), model: z.string() })`
    *   **Output Schema**: `z.object({ response: z.string() })`
    *   **Logic**: This flow will be simpler. It should take the system instruction, user prompt, and model name, and call `ai.generate()` with these parameters. It should return the model's text response.
    *   Export a wrapper function `runAgent`.
6.  **Update Component Calls**: Refactor the components (`AgentStudio`, `DocumentIntelligence`) that previously used a custom `gemini.ts` file to instead import and call these new server-side Genkit flow functions.

### Milestone 5: Feature Implementation

1.  **Dashboard (`src/components/features/dashboard.tsx`)**:
    *   Build the dashboard UI using ShadCN `<Card>` components with the `.glass-panel` style.
    *   Display the three stat cards: "Total Runs", "Active Agents", and "Avg. Latency". Use mock data.
    *   Implement the two charts using `recharts` (`LineChart` and `BarChart`) and populate them with `MOCK_CHART_DATA`. Use the `useIsMobile` hook to adjust chart margins on smaller screens.

2.  **Agent Studio (`src/components/features/agent-studio.tsx`)**:
    *   Create the two sub-tabs "Run" and "Manage".
    *   **Manage Tab**:
        *   Display a large `<Textarea>` bound to `yamlContent` from the context.
        *   When the textarea content changes (`onChange`), update `yamlContent`. Try to parse the new content with `js-yaml`; if successful, update the `agents` state. If it fails, do nothing, allowing the user to see and fix the invalid YAML.
        *   Implement the "Download YAML", "Upload YAML", and "AI Repair" buttons. The AI Repair button should call the `repairAgentYaml` flow function and update the textarea with the result.
    *   **Run Tab**:
        *   Use a `<Select>` to choose an agent from the `agents` list.
        *   Provide an `<Input>` for an optional model override.
        *   A `<Textarea>` for the user's prompt.
        *   The "Execute" button should call the `runAgent` flow, passing the selected agent's details, the user prompt, and the model override.
        *   Display the result in a read-only `<pre>` block with a copy-to-clipboard button.
        *   Manage `isLoading` state to show a spinner on the button during the API call.

3.  **Document Intelligence (`src/components/features/document-intelligence.tsx`)**:
    *   Implement the mode toggle ("Paste Text" / "Upload File").
    *   For "Upload File" mode, create a drag-and-drop zone that accepts supported file types. When a file is uploaded, read it using `FileReader` and display a preview (image preview for images, file icon for others). Store the file content as a data URI if it's a binary type.
    *   The "Process" button should call the `processDocumentIntelligence` flow, passing the content and its type.
    *   Display the AI's response in a large, editable `<Textarea>`.
    *   Add a `<Slider>` to control the font size of the output textarea.
    *   Add "Download .md" and "Download .txt" buttons to save the output.

## 4. Final Polish

*   Ensure all components are responsive and look good on mobile devices. Use the `useIsMobile` hook where necessary.
*   Verify that all UI text is correctly internationalized using the `getLabel` function.
*   Check that loading states are handled gracefully across the app (e.g., spinners on buttons).
*   Ensure toasts are used for notifications like API key errors, file upload success/failure, and YAML repair status.
*   Thoroughly review the application for a cohesive and polished user experience, paying special attention to the "glass panel" aesthetic and the seamless transitions between painter styles.
