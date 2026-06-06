================================================================================
                           CLINE DETAILED GUIDE
================================================================================

TABLE OF CONTENTS
───────────────────────────────────────────────────────────────────────────────
1. What is Cline?
2. Installation
3. API Provider Setup
4. Custom Instructions & Modes
5. Slash Commands
6. Checkpoints
7. .clinerules Custom Instructions
8. MCP Server Integration
9. Tool Permissions
10. File Operations
11. Terminal Execution
12. Web Search Capability
13. Best Practices

================================================================================
1. WHAT IS CLINE?
================================================================================

Cline (formerly Claude Dev) is an autonomous AI coding assistant that operates
directly inside your IDE (VS Code, Cursor, Windsurf, etc.). Unlike chat-only
AI tools, Cline can:

  • Read and write files in your workspace.
  • Execute terminal commands and see output.
  • Search your codebase with regex patterns.
  • Access the web for documentation and APIs.
  • Integrate with MCP (Model Context Protocol) servers.
  • Create and manage its own checkpoints.

================================================================================
2. INSTALLATION
================================================================================

2.1 VS Code
───────────────────────────────────────────────────────────────────────────────
  1. Open VS Code.
  2. Go to Extensions (Ctrl+Shift+X).
  3. Search "Cline" (by Sahil Lavingia & Cline Bot).
  4. Click Install.
  5. After install, the Cline icon appears in the activity bar (or press
     Ctrl+Alt+I to open the chat panel).

Alternatively, install from the marketplace URL:
  https://marketplace.visualstudio.com/items?itemName=saoudrizwan.claude-dev

2.2 Cursor
───────────────────────────────────────────────────────────────────────────────
  Cursor has built-in Cline integration. Enable it via:

================================================================================
3. API PROVIDER SETUP
================================================================================

Cline requires an LLM API provider to function. You configure this in the Cline
panel's settings (gear icon).

3.1 OpenAI
───────────────────────────────────────────────────────────────────────────────
  • Models: gpt-4o, gpt-4o-mini, gpt-4-turbo, o3-mini, o1
  • API Key: From https://platform.openai.com/api-keys
  • Base URL: https://api.openai.com/v1
  • Setup: Select "OpenAI" as provider, paste your API key.

3.2 Anthropic (Original Provider)
───────────────────────────────────────────────────────────────────────────────
  • Models: claude-sonnet-4-20250514, claude-3-5-sonnet-20241022,
            claude-3-opus-20240229, claude-3-haiku-20240307
  • API Key: From https://console.anthropic.com/
  • Base URL: https://api.anthropic.com
  • Setup: Select "Anthropic" as provider, paste API key.
  • Note: Anthropic was the original/default provider for Cline.

3.3 Google Gemini
───────────────────────────────────────────────────────────────────────────────
  • Models: gemini-2.0-flash, gemini-2.5-pro, gemini-1.5-pro, gemini-1.5-flash
  • API Key: From https://aistudio.google.com/apikey
  • Base URL: https://generativelanguage.googleapis.com/v1beta/openai/
  • Setup: Select "OpenAI Compatible" and enter the Gemini base URL.

3.4 OpenRouter
───────────────────────────────────────────────────────────────────────────────
  Provides access to many models through a single API.
  • API Key: From https://openrouter.ai/keys
  • Base URL: https://openrouter.ai/api/v1
  • Setup: Select "OpenAI Compatible", set base URL, paste key.
  • Models available: All major providers (OpenAI, Anthropic, Google, Meta,
    Mistral, DeepSeek, etc.)

3.5 Local Models (Ollama)
───────────────────────────────────────────────────────────────────────────────
  Run models locally on your machine.
  1. Install Ollama from https://ollama.com
  2. Pull a model: ollama pull llama3.2:3b (or codellama, deepseek-coder, etc.)
  3. Cline setup:
     - Provider: "Ollama" (or "OpenAI Compatible")
     - Base URL: http://localhost:11434/v1
     - Model: Select from available models

3.6 Local Models (LM Studio)
───────────────────────────────────────────────────────────────────────────────
  1. Download & install LM Studio from https://lmstudio.ai
  2. Load a model and start the local inference server.
  3. Cline setup:

================================================================================
4. CUSTOM INSTRUCTIONS & MODES
================================================================================

Cline supports three operating modes, each with a distinct system prompt.

4.1 Modes
───────────────────────────────────────────────────────────────────────────────
  • Code Mode: Full access to all tools. Can read, write, execute terminal
    commands, and make file edits. Default behavior.

  • Architect Mode: Focused on planning and design. Can read files and
    search the codebase but typically restricted from writing files or
    executing destructive commands. Best for architecture planning before
    writing code.

  • Ask Mode: Read-only Q&A mode. Can read files and search the codebase
    but cannot write files or execute commands. Best for understanding
    existing code, documentation questions, and code review.

4.2 Switching Modes
───────────────────────────────────────────────────────────────────────────────
  • Use the mode selector dropdown in the Cline panel.
  • Or use slash commands: /code, /architect, /ask

4.3 Custom Instructions
───────────────────────────────────────────────────────────────────────────────
You can add custom instructions that Cline will follow in every conversation.
Access: Cline settings → Custom Instructions

================================================================================
5. SLASH COMMANDS
================================================================================

Cline supports slash commands for quick actions within the chat panel.

5.1 Built-in Commands
───────────────────────────────────────────────────────────────────────────────
  /code         — Switch to Code mode.
  /architect    — Switch to Architect mode.
  /ask          — Switch to Ask mode.
  /help         — Show available commands.
  /clear        — Clear the current conversation.

5.2 Custom Slash Commands
───────────────────────────────────────────────────────────────────────────────
You can define custom slash commands in Cline settings. Each command maps to
a prompt template that gets inserted into the chat.

Example custom commands:
  /fix          → "Review the code for bugs and fix any issues found."
  /test         → "Write unit tests for the selected code."
  /review       → "Perform a thorough code review of the selected code."
  /doc          → "Generate documentation for the code."
  /explain      → "Explain how this code works in simple terms."

Configuration format (in Cline settings):
  [
    { "name": "fix", "prompt": "Review the code for bugs and fix issues." },
    { "name": "test", "prompt": "Write unit tests for the selected code." }
  ]

================================================================================
6. CHECKPOINTS

================================================================================
7. .clinerules CUSTOM INSTRUCTIONS
================================================================================

The .clinerules file allows you to define project-specific instructions that
Cline will automatically follow when working in that project.

7.1 Location & Format
───────────────────────────────────────────────────────────────────────────────
  • File: .clinerules at the project root (same level as package.json).
  • Format: Plain text or markdown.
  • Cline reads this file at the start of each conversation and injects its
    content into the system prompt.

7.2 Example .clinerules
───────────────────────────────────────────────────────────────────────────────
  # Project: MyApp
  # Tech Stack: Next.js 14, TypeScript, Prisma, Tailwind CSS

  ## Coding Standards
  - Use `app/` directory (App Router), not `pages/`.
  - All API routes must include error handling with try/catch.
  - Use `zod` for request validation in API routes.
  - Prefer server components by default; add 'use client' only when needed.

  ## Testing
  - Run `npm run test` before closing any task.
  - Tests use Vitest + React Testing Library.

  ## Commits
  - Follow conventional commits: feat:, fix:, chore:, docs:, refactor:.

  ## Environment
  - .env.local contains local secrets. Never commit it.
  - Ensure DATABASE_URL is set before running migrations.

7.3 Priority
───────────────────────────────────────────────────────────────────────────────
  .clinerules instructions have higher priority than global custom instructions
  but lower priority than explicit user requests in the current conversation.

================================================================================
8. MCP SERVER INTEGRATION
================================================================================

MCP (Model Context Protocol) is an open standard that lets Cline connect to
external tools and data sources through a standardized server interface.

8.1 What MCP Provides
───────────────────────────────────────────────────────────────────────────────
  • Database access: Query databases directly.
  • File system: Additional file operations beyond built-in tools.
  • Web APIs: Interact with GitHub, Jira, Slack, etc.
  • Custom tools: Execute specialized business logic.

================================================================================
9. TOOL PERMISSIONS
================================================================================

Cline's tool permissions control what actions Cline can perform and whether it
needs your approval before executing them.

9.1 Permission Levels
───────────────────────────────────────────────────────────────────────────────
  • Allow: Cline can use the tool without asking.
  • Ask: Cline requests your approval each time (default for most tools).
  • Deny: Cline cannot use the tool.

9.2 Configurable Tools
───────────────────────────────────────────────────────────────────────────────
  File Operations:
    Read files         — Allow / Ask / Deny
    Write files        — Allow / Ask / Deny
    Edit files         — Allow / Ask / Deny
    Delete files       — Allow / Ask / Deny
    Search codebase    — Allow / Ask / Deny

  Terminal:
    Execute commands   — Allow / Ask / Deny

  Network:
    Web fetch          — Allow / Ask / Deny

  MCP Tools:
    Each MCP server's tools — Allow / Ask / Deny per tool

9.3 Configuration
───────────────────────────────────────────────────────────────────────────────
  Access: Cline settings → Tool Permissions
  Changes take effect immediately. No restart needed.

9.4 Recommended Settings
───────────────────────────────────────────────────────────────────────────────
  • For trusted projects: Allow read/search, Ask for write/edit/terminal.
  • For sensitive projects: Ask for everything.
  • For automated/CI: Allow everything (use with caution).
  • Read and Search can safely be set to Allow — they're non-destructive.

================================================================================

================================================================================
11. TERMINAL EXECUTION
================================================================================

Cline can execute shell commands in your workspace and see their output.

11.1 How It Works
───────────────────────────────────────────────────────────────────────────────
  • Cline spawns a shell process (bash on Linux/macOS, PowerShell on Windows).
  • Commands execute from the workspace root.
  • Cline sees stdout, stderr, and the exit code.
  • You see live output in the Cline panel's terminal view.

11.2 What Cline Can Do
───────────────────────────────────────────────────────────────────────────────
  • Run build tools: npm run build, cargo build, go build
  • Execute tests: npm test, pytest, cargo test
  • Install dependencies: npm install, pip install
  • Run linters/formatters: eslint, prettier, rustfmt
  • Git operations: git add, git commit, git push
  • Start dev servers: npm run dev
  • Any command-line tool available in your PATH

11.3 Safety & Limitations
───────────────────────────────────────────────────────────────────────────────
  • Long-running commands may time out.
  • Interactive commands (requiring stdin) are not supported.
  • Commands that modify system files outside the project are blocked.
  • Terminal execution is subject to tool permission settings.
  • Cline cannot persist state between commands (no shell sessions).

11.4 Best Practices
───────────────────────────────────────────────────────────────────────────────
  • Use quiet flags where possible (--silent, -q) to reduce output noise.
  • Break complex operations into smaller, focused commands.
  • Avoid destructive commands (rm -rf, sudo, etc.) without review.
  • Use --dry-run flags to preview changes before executing.

================================================================================
12. WEB SEARCH CAPABILITY
───────────────────────────────────────────────────────────────────────────────

Cline can fetch and analyze web content, enabling it to access documentation,
APIs, and online resources.

12.1 How Web Access Works
───────────────────────────────────────────────────────────────────────────────
  • Cline's fetch_web_content tool sends HTTP requests to URLs.
  • It retrieves the content and provides an analysis prompt to the LLM.
  • The LLM processes the fetched content according to the prompt.

12.2 Use Cases
───────────────────────────────────────────────────────────────────────────────
  • Reading API documentation: "Fetch the Stripe API docs for creating
    a payment intent."
  • Checking package versions: "What is the latest version of Next.js?"
  • Looking up error solutions: "What does this error mean? (paste error)"
  • Researching best practices: "Fetch the React 19 migration guide."
  • Reading RFCs and specifications.

12.3 Limitations
───────────────────────────────────────────────────────────────────────────────
  • Cannot execute JavaScript on pages (no browser engine).
  • May not render SPAs (single-page applications) correctly.
  • Some sites may block automated requests.
  • Large pages may be truncated.
  • Requires tool permission: "Allow" or "Ask" for web fetch.

================================================================================
13. BEST PRACTICES
───────────────────────────────────────────────────────────────────────────────

13.1 Prompting Guidelines
───────────────────────────────────────────────────────────────────────────────
  • Be specific and explicit about what you want.
  • Provide context: tech stack, file locations, constraints.
  • Break large tasks into smaller, focused requests.
  • Use slash commands to quickly switch contexts.
  • Reference specific files when possible.

13.2 Project Setup
───────────────────────────────────────────────────────────────────────────────
  • Always create a .clinerules file for project-specific instructions.
  • Configure global custom instructions for personal preferences.
  • Set appropriate tool permissions for your workflow.
  • Enable checkpoints for safety.

13.3 Workflow Optimization
───────────────────────────────────────────────────────────────────────────────
  • Start with /architect mode to plan complex changes.
  • Switch to /code mode for implementation.
  • Use /ask mode for quick questions without risking changes.
  • Leverage MCP servers for database and external API access.
  • Use search_codebase instead of reading large files.

13.4 Security
───────────────────────────────────────────────────────────────────────────────
  • Never expose API keys in prompts or files that Cline can read.
  • Review all terminal commands before execution.
  • Be cautious with force-push and destructive Git commands.
  • Audit MCP server configurations regularly.
  • Use "Ask" permission for write and terminal operations.

13.5 Troubleshooting
───────────────────────────────────────────────────────────────────────────────
  • If Cline is not responding: Check API key validity and quota.
  • If commands fail: Verify the tool is installed in your PATH.
  • If checkpoints are missing: Ensure Git is initialized in the project.
  • If MCP servers fail: Check logs in Cline settings → MCP Servers.

================================================================================
END OF CLINE DETAILED GUIDE
================================================================================

10. FILE OPERATIONS
================================================================================

Cline has rich file manipulation capabilities through its built-in tools.

10.1 Available File Tools
───────────────────────────────────────────────────────────────────────────────
  read_files       — Read full content or specific line ranges of files.
  editor           — Edit files by replacing text, inserting at line, or
                     creating new files.
  search_codebase  — Search files using regex patterns (like grep).
  write_to_file    — Create or overwrite files with full content.

10.2 Capabilities
───────────────────────────────────────────────────────────────────────────────
  • Read any text file in the workspace.
  • Read binary files supported as images.
  • Edit files with surgical precision (find-and-replace).
  • Insert content at specific lines.
  • Create new files with full content.
  • Search across the entire codebase with regex.
  • List directory contents (via terminal).

10.3 Limitations
───────────────────────────────────────────────────────────────────────────────
  • Cannot read binary files (except images).
  • Large files (>300 lines) should be searched rather than fully read.
  • File operations are subject to tool permission settings.
  • Writes outside the workspace directory are blocked.


8.2 Setting Up MCP Servers
───────────────────────────────────────────────────────────────────────────────
Configuration is done in Cline settings → MCP Servers.

Each MCP server is defined by:
  • Name: Unique identifier.
  • Command: The executable to run (node, python, npx, etc.).
  • Arguments: Command-line arguments.
  • Environment variables: Key-value pairs for config.

Example: PostgreSQL MCP Server
  Name: postgres
  Command: npx
  Args: ["-y", "@anthropic/mcp-postgres"]
  Env: { "DATABASE_URL": "postgresql://user:pass@localhost:5432/mydb" }

Example: Filesystem MCP Server
  Name: filesystem
  Command: npx
  Args: ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/allowed"]
  Env: {}

8.3 Available MCP Servers (Community & Official)
───────────────────────────────────────────────────────────────────────────────
  • @modelcontextprotocol/server-filesystem   — Sandboxed file access
  • @modelcontextprotocol/server-github        — GitHub API
  • @modelcontextprotocol/server-git           — Git operations
  • @anthropic/mcp-postgres                    — PostgreSQL queries
  • @anthropic/mcp-slack                       — Slack integration
  • @anthropic/mcp-sqlite                      — SQLite queries
  • @getcursor/mcp                            — Cursor-specific tools

8.4 Security
───────────────────────────────────────────────────────────────────────────────
  • Each MCP server runs as a subprocess.
  • Permission to invoke MCP tools is controlled by Cline's tool permission
    settings.
  • MCP servers only have access to what you explicitly configure.
  • Review the source code of community MCP servers before using them.

================================================================================

Cline can automatically create checkpoints (snapshots of your workspace) before
making changes. This allows rolling back if something goes wrong.

6.1 How Checkpoints Work
───────────────────────────────────────────────────────────────────────────────
  • Cline uses Git behind the scenes.
  • Before each file write/edit/delete operation, Cline creates a Git commit
    on a special checkpoint branch.
  • Checkpoints appear in the Cline panel timeline.

6.2 Managing Checkpoints
───────────────────────────────────────────────────────────────────────────────
  • Enable/disable: Cline settings → Checkpoints.
  • View timeline: Click the clock icon in Cline panel.
  • Restore: Click a checkpoint entry to revert workspace to that state.
  • Checkpoints are local Git operations — they don't push to remotes.

6.3 Best Practices
───────────────────────────────────────────────────────────────────────────────
  • Keep checkpoints enabled (they're lightweight Git operations).
  • Use descriptive prompts so checkpoints are meaningful.
  • If you use your own Git workflow, know that checkpoint branches are
    separate and won't interfere with your main workflow.


Example custom instructions:
  "Always use TypeScript with strict mode. Write JSDoc comments for all
   exported functions. Use functional components with hooks, never classes.
   Run 'npm run typecheck' after every change."

Custom instructions are appended to Cline's system prompt and are persistent
across sessions.

     - Provider: "OpenAI Compatible"
     - Base URL: http://localhost:1234/v1
     - API Key: (not needed, can be any placeholder)
     - Model: Enter the model name manually

3.7 OpenAI Compatible (Generic)
───────────────────────────────────────────────────────────────────────────────
Any API that implements the OpenAI chat completions endpoint format:

  Provider: "OpenAI Compatible"
  Base URL: <your-endpoint>/v1
  API Key:  <your-key>
  Model:    <model-id>

  Settings → Features → Cline → Enable

2.3 Windsurf
───────────────────────────────────────────────────────────────────────────────
  Install from the Windsurf extension marketplace, searching for "Cline".
  Configure similarly to VS Code.

2.4 Post-Install Verification
───────────────────────────────────────────────────────────────────────────────
  • Open the Cline panel (Cline icon in sidebar).
  • You should see a chat interface with a model selector dropdown.
  • If the panel shows "No provider configured", proceed to API setup.


Cline uses a "think then act" loop: it plans, executes tool calls, observes
results, and iterates. This makes it far more capable than simple chat bots.

Key architectural details:
  • Cline runs locally in your IDE as an extension.
  • API calls go to the provider you configure (OpenAI, Anthropic, etc.).
  • Your code never leaves your machine unless sent to the LLM API.
  • Cline maintains a conversation context window for each session.
  • It uses a system prompt + custom instructions to shape behavior.

