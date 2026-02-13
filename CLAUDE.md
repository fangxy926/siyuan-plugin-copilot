# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is **siyuan-plugin-copilot**, a SiYuan Note (思源笔记) plugin that provides AI assistant functionality. It supports multiple AI providers (OpenAI, Gemini, DeepSeek, Claude, etc.) and offers three chat modes: ask (Q&A), edit (note editing), and agent (tool-based automation).

## Tech Stack

- **Build Tool**: Vite 5.x
- **Frontend Framework**: Svelte 4.x
- **Language**: TypeScript
- **Styling**: SCSS/Sass
- **Package Manager**: pnpm
- **Target Platform**: SiYuan Note plugin environment

## Common Development Commands

```bash
# Install dependencies
pnpm install

# Development build with watch mode and auto-copy to SiYuan plugins directory
pnpm run dev

# Production build (outputs to dist/)
pnpm run build

# Build and create installable package.zip
pnpm run make-install

# Manually copy dev build to SiYuan plugins directory
node scripts/make_dev_copy.js
```

## Project Structure

### Core Files

- **src/index.ts** - Plugin entry point. Main plugin class `PluginSample` that initializes the plugin, registers commands, and manages the AI sidebar and chat dialogs.
- **src/api.ts** - SiYuan API wrapper. Contains all API calls to interact with SiYuan notes (blocks, notebooks, SQL queries, file operations, database operations).
- **src/ai-chat.ts** - AI chat API module. Handles streaming chat requests, supports multiple providers (OpenAI-compatible, Gemini native, Claude native), image generation, and thinking/reasoning modes.
- **src/tools/index.ts** - Agent mode tool implementations. Defines available tools for AI to interact with SiYuan (SQL queries, block operations, document management, database operations).

### Settings & State

- **src/defaultSettings.ts** - Default settings structure including AI provider configurations, model presets, and UI preferences.
- **src/stores/settings.ts** - Settings store for managing plugin configuration persistence.

### UI Components (Svelte)

- **src/ai-sidebar.svelte** - Main AI chat sidebar interface
- **src/SettingsPannel.svelte** - Plugin settings panel
- **src/components/** - Various UI components:
  - `ChatDialog.svelte` - Chat dialog window
  - `MultiModelSelector.svelte` - Multi-model selection UI
  - `ModelPreset.svelte` - Model preset management
  - `SessionManager.svelte` - Chat session management
  - `ToolSelector.svelte` - Agent mode tool selection
  - `TranslateDialog.svelte` - Translation dialog
  - `WebAppManager.svelte` - Web app/mini-program management
  - `ProviderConfigPanel.svelte` - AI provider configuration

### Utilities

- **src/utils/i18n.ts** - Internationalization utilities
- **src/utils/hotkey.ts** - Keyboard shortcut handling
- **src/utils/modelCapabilities.ts** - Model capability detection
- **src/utils/assets.ts** - Asset file handling
- **src/utils/webParser.ts** - Web page content parsing

## Architecture Notes

### AI Provider Support

The plugin supports multiple AI providers through a unified interface in `ai-chat.ts`:

- **OpenAI-compatible**: DeepSeek, Moonshot, Volcano Engine, and custom OpenAI-compatible APIs
- **Gemini native**: Google Gemini API with native format
- **Claude native**: Anthropic Claude API with native format

Each provider has a configuration in `PROVIDER_CONFIGS` with base URL, endpoints, and authentication headers.

### Chat Modes

1. **Ask Mode** - Standard Q&A with optional multi-model simultaneous responses
2. **Edit Mode** - AI proposes edits to note content with diff view and undo support
3. **Agent Mode** - AI can use tools to query and modify notes autonomously

### Agent Tools

Tools are defined in `src/tools/index.ts` and include:
- `siyuan_sql_query` - Execute SQL queries on SiYuan database
- `siyuan_get_block_content` - Retrieve block content
- `siyuan_insert_block` - Insert new blocks
- `siyuan_update_block` - Update existing blocks
- `siyuan_create_document` - Create new documents
- `siyuan_list_notebooks` - List all notebooks
- `siyuan_database` - Database (AttributeView) operations
- And more for document management

### Development Workflow

During development (`pnpm run dev`):
1. Vite builds to `dev/` directory with source maps
2. `make_dev_copy.js` automatically copies files to the configured SiYuan plugins directory
3. The hardcoded target path in `scripts/make_dev_copy.js` is `D:\Notes\Siyuan\Achuan-2\data\plugins`

For production:
1. `pnpm run build` outputs to `dist/`
2. `vite-plugin-zip-pack` creates `package.zip` for distribution
3. Static files (README, plugin.json, i18n, assets) are copied to output

### Key Type Definitions

- **BlockId, DocumentId, NotebookId** - SiYuan entity identifiers
- **Message** - Chat message structure supporting text, images, tool calls, and multi-model responses
- **Tool, ToolCall, ToolResult** - Agent mode tool definitions
- **AIProvider** - Supported AI provider types

## Development Notes

- The plugin uses SiYuan's `siyuan` package for core API access (imported as external)
- Settings are persisted to SiYuan's storage via ` SETTINGS_FILE = "settings.json"`
- The plugin registers dock panels, commands, and top-bar buttons through the SiYuan Plugin API
- Drag-and-drop support is implemented for dragging note blocks into the chat interface
- Image handling supports blob URLs converted to base64 for API transmission
