================================================================================
                    CLINE + TELEGRAM BOT DETAILED GUIDE
================================================================================

TABLE OF CONTENTS
───────────────────────────────────────────────────────────────────────────────
1. Overview
2. Setting Up a Telegram Bot (BotFather)
3. Getting a Bot Token
4. Library Selection
5. Creating a Bridge — Architecture
6. Handling Commands
7. Session Management
8. File Handling
9. Webhook vs Polling Setup
10. Deployment Considerations
11. Security Best Practices
12. Full Example Implementation

================================================================================
1. OVERVIEW
───────────────────────────────────────────────────────────────────────────────

Connecting Cline to Telegram allows you to interact with your AI coding
assistant from anywhere via Telegram messages. This bridge enables:

  • Sending coding tasks from your phone.
  • Receiving code reviews and explanations on the go.
  • Triggering Cline operations remotely.
  • Sharing code snippets and files via Telegram for Cline to process.
  • Getting notifications when long-running tasks complete.

Architecture:
  Telegram App ←→ Telegram Bot API ←→ Your Bridge Server ←→ Cline API

The bridge server acts as middleware, translating Telegram messages into Cline
API calls and relaying responses back to Telegram.

Key design decisions:
  • The bridge runs as a standalone service (Node.js or Python).
  • Cline runs in your IDE or as a headless instance.
  • Communication with Cline happens via its API or filesystem.

================================================================================
2. SETTING UP A TELEGRAM BOT (BOTFATHER)
───────────────────────────────────────────────────────────────────────────────

Telegram bots are created and managed through BotFather, Telegram's official
bot management system.

2.1 Creating a New Bot
───────────────────────────────────────────────────────────────────────────────
  1. Open Telegram and search for @BotFather (official bot).
  2. Start a chat and send: /newbot

================================================================================
3. GETTING A BOT TOKEN
───────────────────────────────────────────────────────────────────────────────

The bot token is the sole authentication credential for your Telegram bot.

3.1 Token Format
───────────────────────────────────────────────────────────────────────────────
  <bot_id>:<hash>

  Example: 7234567890:AAHdqTcvCH1vGWJxfSeofSAs0K5PALDsaw

  • Bot ID: Numeric, identifies your bot.
  • Hash: Cryptographic hash that authenticates API calls.

3.2 Where to Find It
───────────────────────────────────────────────────────────────────────────────
  • After creating the bot with BotFather, the token is displayed once.
  • If you lose it, send /token to BotFather to regenerate.
  • Regenerating invalidates the old token immediately.

3.3 Storing the Token
───────────────────────────────────────────────────────────────────────────────
  • Never hardcode the token in source code.
  • Use environment variables:
    Windows PowerShell:   $env:TELEGRAM_BOT_TOKEN = "723..."
    Linux/macOS:          export TELEGRAM_BOT_TOKEN="723..."
    .env file:            TELEGRAM_BOT_TOKEN=723...


4.2 Python — python-telegram-bot (v20+)
───────────────────────────────────────────────────────────────────────────────
  Install: pip install python-telegram-bot

  Features:
    • Asynchronous (asyncio-based in v20+).
    • Comprehensive API coverage.
    • Application and handler architecture.
    • Conversation handler for multi-step interactions.
    • Rate limiting built-in.

  Basic setup:
    from telegram import Update
    from telegram.ext import Application, CommandHandler, MessageHandler, filters

    app = Application.builder().token(TOKEN).build()
    app.add_handler(CommandHandler("start", start_command))
    app.run_polling()

4.3 Library Comparison
───────────────────────────────────────────────────────────────────────────────
  Feature               node-telegram-bot-api   python-telegram-bot
  ─────────────────────────────────────────────────────────────────────
  Language              JavaScript/TypeScript   Python 3.8+
  Async model           Callbacks/Promises      asyncio (native)
  Polling               Built-in                Built-in
  Webhook               Manual setup            Built-in
  File handling         Built-in                Built-in
  Conversation states   Manual                  Built-in handler
  Community size        Large                   Very large
  Learning curve        Low                     Medium

  Recommendation: Use python-telegram-bot for Python shops or complex
  conversation flows. Use node-telegram-bot-api for JavaScript shops
  or simpler polling-based bots.

================================================================================
5. CREATING A BRIDGE — ARCHITECTURE
───────────────────────────────────────────────────────────────────────────────

5.1 High-Level Architecture
───────────────────────────────────────────────────────────────────────────────
  ┌──────────┐     Telegram API      ┌──────────────┐     HTTP      ┌────────────────┐
  │ Telegram │  ◄─────────────────►  │  Bridge App  │  ◄────────►  │  Cline/LLM API │
  │  Client  │                       │ (Bot Server) │               │                │
  └──────────┘                       └──────────────┘               └────────────────┘

5.2 Message Flow
───────────────────────────────────────────────────────────────────────────────
  1. User sends message to bot on Telegram.
  2. Bridge receives update (polling or webhook).
  3. Bridge parses: detects command, identifies user/session,
     extracts text and files.
  4. Bridge formats request for Cline/LLM API.

================================================================================
6. HANDLING COMMANDS
───────────────────────────────────────────────────────────────────────────────

6.1 Defining Commands
───────────────────────────────────────────────────────────────────────────────
Register with BotFather via /setcommands:

  /ask    - Ask Cline a question (Ask mode)
  /code   - Execute a coding task (Code mode)
  /chat   - Free-form conversation
  /status - Check bot connection status
  /help   - Show available commands

6.2 /start Handler (Node.js)
───────────────────────────────────────────────────────────────────────────────
  bot.onText(/\/start/, async (msg) => {
    const welcome = `*Cline Telegram Bridge* 🤖\n\n`
      + `I connect you to your AI coding assistant.\n\n`
      + `/ask <question> - Ask a coding question\n`
      + `/code <task>    - Execute a coding task\n`
      + `/chat <message> - Free-form chat\n`
      + `/help           - Show this message`;

    await bot.sendMessage(msg.chat.id, welcome, { parse_mode: 'Markdown' });
  });

6.3 /ask Handler
───────────────────────────────────────────────────────────────────────────────
  bot.onText(/\/ask (.+)/, async (msg, match) => {
    const question = match[1];
    const chatId = msg.chat.id;
    bot.sendChatAction(chatId, 'typing');

    try {
      const response = await clineService.ask(question);
      await bot.sendMessage(chatId, response, { parse_mode: 'Markdown' });
    } catch (error) {
      await bot.sendMessage(chatId, `Error: ${error.message}`);
    }
  });

6.4 /code Handler
───────────────────────────────────────────────────────────────────────────────
  bot.onText(/\/code (.+)/, async (msg, match) => {
    const task = match[1];
    const chatId = msg.chat.id;
    bot.sendChatAction(chatId, 'typing');
    bot.sendMessage(chatId, "Processing your coding task... ⏳");

    try {
      const response = await clineService.code(task);
      const formatted = formatCodeResponse(response);
      await bot.sendMessage(chatId, formatted, {
        parse_mode: 'Markdown',
        disable_web_page_preview: true
      });
    } catch (error) {
      await bot.sendMessage(chatId, `Coding error: ${error.message}`);

================================================================================
7. SESSION MANAGEMENT
───────────────────────────────────────────────────────────────────────────────

Sessions maintain conversation context per user across multiple messages.

7.1 Session Data Structure
───────────────────────────────────────────────────────────────────────────────
  {
    userId: 123456789,
    context: [
      { role: "user", content: "..." },
      { role: "assistant", content: "..." }
    ],
    defaultMode: "chat",
    createdAt: 1700000000,
    lastActiveAt: 1700000100,
    messageCount: 5,
    tokenCount: 1500
  }

7.2 Storage Options
───────────────────────────────────────────────────────────────────────────────
  • In-memory: Simple, lost on restart. Good for dev.
  • SQLite: Persistent, single-file. Good for small deployments.
  • Redis: Fast, TTL expiry. Good for production.
  • JSON files: Simple persistence for low traffic.

7.3 Session Cleanup
───────────────────────────────────────────────────────────────────────────────
  • Max context window (e.g., last 20 messages).
  • Session expiry (e.g., 24h inactivity).
  • Trim oldest messages when context exceeds limits.
  • Track total token usage to prevent runaway costs.

7.4 Python Example
───────────────────────────────────────────────────────────────────────────────
  class SessionManager:
      def __init__(self, max_context=20):
          self.sessions = {}
          self.max_context = max_context

      def get_or_create(self, user_id):
          if user_id not in self.sessions:
              self.sessions[user_id] = {
                  'context': [], 'mode': 'chat', 'created_at': time.time()
              }
          return self.sessions[user_id]

      def add_message(self, user_id, role, content):
          s = self.get_or_create(user_id)
          s['context'].append({'role': role, 'content': content})
          if len(s['context']) > self.max_context:
              s['context'] = s['context'][-self.max_context:]

================================================================================
9. WEBHOOK vs POLLING SETUP
───────────────────────────────────────────────────────────────────────────────

Telegram bots receive updates via two mechanisms: polling and webhooks.

9.1 Polling
───────────────────────────────────────────────────────────────────────────────
The bot periodically requests updates from Telegram's API.

  Pros:
    • Simple to set up — no public URL needed.
    • Works behind NAT, firewalls, or localhost.
    • Good for development and personal bots.

  Cons:
    • Increases latency (up to polling interval).
    • Less efficient (repeated requests even when no updates).
    • Doesn't scale well to high traffic.

  Node.js polling:
    const bot = new TelegramBot(TOKEN, { polling: true });

  Python polling:
    app.run_polling()

9.2 Webhook
───────────────────────────────────────────────────────────────────────────────
Telegram sends updates to your server via POST requests to a public URL.

  Pros:
    • Real-time updates (no polling delay).
    • More efficient (push-based).
    • Scales better for production.

  Cons:
    • Requires a public HTTPS URL with valid SSL certificate.
    • More complex infrastructure.
    • Only one webhook per bot.

  Setting a webhook (Node.js):
    bot.setWebHook('https://your-domain.com/webhook');

    // Or with Express:
    const express = require('express');
    const app = express();
    app.use(express.json());

    app.post('/webhook', (req, res) => {
      bot.processUpdate(req.body);
      res.sendStatus(200);
    });

    bot.setWebHook('https://your-domain.com/webhook');

  Python webhook (using Flask):
    @app.route('/webhook', methods=['POST'])
    def webhook():
        update = Update.de_json(request.get_json(), app.bot)
        app.dispatcher.process_update(update)
        return 'ok', 200

9.3 Choosing Between Polling and Webhook
───────────────────────────────────────────────────────────────────────────────
  Use polling if:
    • The bot runs on your local machine.
    • You don't have a public domain/static IP.
    • This is a personal/development bot.
    • You want the simplest setup.

  Use webhook if:
    • The bot runs on a cloud server (AWS, DigitalOcean, etc.).
    • You need real-time response times.
    • The bot serves multiple users.
    • You want production-ready architecture.

9.4 Setting up a Public URL for Webhook
───────────────────────────────────────────────────────────────────────────────
  Option A — VPS with Nginx reverse proxy:
    server {
        listen 443 ssl;
        server_name your-domain.com;
        location /webhook {
            proxy_pass http://localhost:PORT;
        }
    }

  Option B — ngrok (development only):
    ngrok http 3000
    # Sets webhook to https://xxxx.ngrok.io/webhook

  Option C — Cloudflare Tunnel:
    cloudflared tunnel --url http://localhost:3000

  Option D — Railway/Render/Heroku:
    Most platforms provide HTTPS URLs automatically.

================================================================================
10. DEPLOYMENT CONSIDERATIONS
───────────────────────────────────────────────────────────────────────────────

10.1 Process Management
───────────────────────────────────────────────────────────────────────────────
  • Use a process manager (PM2, systemd, supervisor) to keep the bot alive.

================================================================================
11. SECURITY BEST PRACTICES
───────────────────────────────────────────────────────────────────────────────

11.1 User Authorization
───────────────────────────────────────────────────────────────────────────────
  const ALLOWED_USERS = process.env.ALLOWED_USER_IDS?.split(',') || [];

  function isAuthorized(userId) {
    if (ALLOWED_USERS.length === 0 || !ALLOWED_USERS[0]) return true;
    return ALLOWED_USERS.includes(String(userId));
  }

11.2 API Key Protection
───────────────────────────────────────────────────────────────────────────────
  • Store secrets in environment variables or a secrets manager.
  • Never expose API keys in bot responses or logs.
  • Use .env files with .gitignore. Rotate keys periodically.

11.3 Input Validation
───────────────────────────────────────────────────────────────────────────────
  • Sanitize all user input before sending to LLM APIs.
  • Limit message length (max 4000 chars per message).
  • Reject messages with suspicious patterns.
  • Validate file types before processing.

11.4 Rate Limiting
───────────────────────────────────────────────────────────────────────────────
  • Per-user rate limit (e.g., 10 messages/user/minute).
  • Daily token limit per user.
  • Return clear error when limits exceeded.

  Node.js rate limiter:
    const rateLimit = new Map();
    function checkRateLimit(userId) {
      const now = Date.now();
      const LIMIT = { max: 10, window: 60000 };
      const user = rateLimit.get(userId) || { count: 0, resetAt: now + LIMIT.window };
      if (now > user.resetAt) { user.count = 0; user.resetAt = now + LIMIT.window; }
      user.count++;
      rateLimit.set(userId, user);
      return user.count <= LIMIT.max;
    }


================================================================================
12. FULL EXAMPLE IMPLEMENTATION
───────────────────────────────────────────────────────────────────────────────

Below is a complete minimal Node.js implementation of a Cline-Telegram bridge.

12.1 package.json
───────────────────────────────────────────────────────────────────────────────
  {
    "name": "cline-telegram-bridge",
    "version": "1.0.0",
    "dependencies": {
      "node-telegram-bot-api": "^0.64.0",
      "openai": "^4.0.0",
      "dotenv": "^16.0.0"
    }
  }

12.2 index.js (Complete Bridge)
───────────────────────────────────────────────────────────────────────────────
  require('dotenv').config();
  const TelegramBot = require('node-telegram-bot-api');
  const OpenAI = require('openai');

  // ── Config ──
  const TOKEN = process.env.TELEGRAM_BOT_TOKEN;
  const ALLOWED_USERS = (process.env.ALLOWED_USER_IDS || '').split(',');

  // ── LLM Client ──
  const llm = new OpenAI({
    apiKey: process.env.LLM_API_KEY,
    baseURL: process.env.LLM_BASE_URL || 'https://api.openai.com/v1'
  });

  // ── Bot Setup ──
  const bot = new TelegramBot(TOKEN, { polling: true });
  const sessions = new Map();

  // ── Auth ──
  function isAuthorized(userId) {
    if (ALLOWED_USERS.length === 0 || !ALLOWED_USERS[0]) return true;
    return ALLOWED_USERS.includes(String(userId));
  }

  // ── LLM Call ──
  async function callLLM(messages, mode = 'chat') {
    const prompts = {
      ask: 'You are a coding assistant. Answer questions concisely.',
      code: 'You are an expert programmer. Write clean, working code.',
      chat: 'You are a helpful AI assistant.'
    };
    const response = await llm.chat.completions.create({
      model: process.env.LLM_MODEL || 'gpt-4o',
      messages: [
        { role: 'system', content: prompts[mode] || prompts.chat },
        ...messages
      ],
      temperature: mode === 'code' ? 0.3 : 0.7,
      max_tokens: 2000
    });
    return response.choices[0].message.content;
  }

  // ── Session ──
  function getSession(userId) {
    if (!sessions.has(userId)) {
      sessions.set(userId, { messages: [], mode: 'chat' });
    }
    return sessions.get(userId);
  }

  // ── Handlers ──
  bot.onText(/\/start|\/help/, (msg) => {
    if (!isAuthorized(msg.from.id)) return;
    bot.sendMessage(msg.chat.id,
      `*Cline Telegram Bridge* 🤖\n\n`
      + `/ask <q>   — Ask a coding question\n`
      + `/code <t>  — Execute a coding task\n`
      + `/chat <m>  — Free-form chat\n`
      + `/clear     — Clear context\n`
      + `/help      — This message`,
      { parse_mode: 'Markdown' });
  });

  bot.onText(/\/ask (.+)/, async (msg, match) => {
    if (!isAuthorized(msg.from.id)) return;
    bot.sendChatAction(msg.chat.id, 'typing');
    try {
      const reply = await callLLM([{ role: 'user', content: match[1] }], 'ask');
      await bot.sendMessage(msg.chat.id, reply, { parse_mode: 'Markdown' });
    } catch (e) {
      await bot.sendMessage(msg.chat.id, `Error: ${e.message}`);
    }
  });

  bot.onText(/\/code (.+)/, async (msg, match) => {
    if (!isAuthorized(msg.from.id)) return;
    bot.sendChatAction(msg.chat.id, 'typing');
    try {
      const reply = await callLLM([{ role: 'user', content: match[1] }], 'code');
      await bot.sendMessage(msg.chat.id, reply, {
        parse_mode: 'Markdown', disable_web_page_preview: true
      });
    } catch (e) {
      await bot.sendMessage(msg.chat.id, `Error: ${e.message}`);
    }
  });

  bot.onText(/\/chat (.+)/, async (msg, match) => {
    if (!isAuthorized(msg.from.id)) return;
    const session = getSession(msg.from.id);
    session.messages.push({ role: 'user', content: match[1] });
    bot.sendChatAction(msg.chat.id, 'typing');
    try {
      const reply = await callLLM(session.messages, 'chat');
      session.messages.push({ role: 'assistant', content: reply });
      await bot.sendMessage(msg.chat.id, reply, { parse_mode: 'Markdown' });
    } catch (e) {
      await bot.sendMessage(msg.chat.id, `Error: ${e.message}`);
    }
  });

  bot.onText(/\/clear/, (msg) => {
    if (!isAuthorized(msg.from.id)) return;
    getSession(msg.from.id).messages = [];
    bot.sendMessage(msg.chat.id, 'Context cleared.');
  });

  // Non-command → /chat
  bot.on('message', async (msg) => {
    if (!msg.text || msg.text.startsWith('/')) return;
    if (!isAuthorized(msg.from.id)) return;
    const session = getSession(msg.from.id);
    session.messages.push({ role: 'user', content: msg.text });
    bot.sendChatAction(msg.chat.id, 'typing');
    try {
      const reply = await callLLM(session.messages, session.mode);
      session.messages.push({ role: 'assistant', content: reply });
      await bot.sendMessage(msg.chat.id, reply, { parse_mode: 'Markdown' });
    } catch (e) {
      await bot.sendMessage(msg.chat.id, `Error: ${e.message}`);
    }
  });

  console.log('Cline Telegram Bridge running...');

================================================================================
END OF CLINE + TELEGRAM BOT DETAILED GUIDE
================================================================================

11.5 Data Privacy
───────────────────────────────────────────────────────────────────────────────
  • Don't log full message content in production.
  • Clear session data after expiry.
  • Inform users messages are sent to LLM APIs.

11.6 HTTPS & SSL
───────────────────────────────────────────────────────────────────────────────
  • Webhooks require HTTPS with valid SSL (Let's Encrypt is free).
  • Validate Telegram's IP range for webhook requests.

11.7 Bot Token Security
───────────────────────────────────────────────────────────────────────────────
  • Regenerate via BotFather if token is compromised.
  • Old token is invalidated instantly.

  • Implement graceful shutdown on SIGTERM/SIGINT.

  PM2 (Node.js):
    pm2 start index.js --name cline-telegram-bridge
    pm2 save
    pm2 startup

  systemd (Linux):
    [Unit]
    Description=Cline Telegram Bridge
    After=network.target

    [Service]
    ExecStart=/usr/bin/node /opt/bridge/index.js
    Restart=always
    EnvironmentFile=/opt/bridge/.env

    [Install]
    WantedBy=multi-user.target

10.2 Logging
───────────────────────────────────────────────────────────────────────────────
  • Log all incoming requests and outgoing responses.
  • Log errors with stack traces.
  • Use structured logging (JSON format) for production.
  • Implement log rotation to prevent disk fill-up.

10.3 Monitoring
───────────────────────────────────────────────────────────────────────────────
  • Health check endpoint: GET /health → { status: "ok", uptime: ... }
  • Monitor API response times.
  • Set up alerts for errors and rate limits.
  • Track token usage daily.

10.4 Scaling
───────────────────────────────────────────────────────────────────────────────
  • Use Redis for session storage to enable horizontal scaling.
  • Consider message queues (RabbitMQ, Redis) for async processing.
  • Implement request queuing for long-running Cline tasks.
  • Use worker threads for parallel processing of multiple user requests.

10.5 Environment Variables Checklist
───────────────────────────────────────────────────────────────────────────────
  TELEGRAM_BOT_TOKEN      — Required: Telegram bot token
  CLINE_API_URL           — Required: Cline/LLM API endpoint
  CLINE_API_KEY           — Required: API key for Cline service
  ALLOWED_USER_IDS        — Optional: Comma-separated list of allowed users
  SESSION_STORAGE         — Optional: memory | sqlite | redis
  REDIS_URL               — Optional: Redis connection string
  LOG_LEVEL               — Optional: debug | info | warn | error
  PORT                    — Optional: Webhook server port (default: 3000)


      def clear_context(self, user_id):
          if user_id in self.sessions:
              self.sessions[user_id]['context'] = []

================================================================================
8. FILE HANDLING
───────────────────────────────────────────────────────────────────────────────

Telegram supports file attachments: code files, images, documents, etc.

8.1 Receiving Files
───────────────────────────────────────────────────────────────────────────────
When a user sends a file, the bot receives a file_id. You must download it:

  // Node.js — Download file from Telegram
  bot.on('document', async (msg) => {
    const fileId = msg.document.file_id;
    const fileName = msg.document.file_name;

    // Get file path from Telegram
    const file = await bot.getFile(fileId);
    const fileUrl = `https://api.telegram.org/file/bot${TOKEN}/${file.file_path}`;

    // Download the file
    const response = await axios.get(fileUrl, { responseType: 'stream' });
    const localPath = `./downloads/${fileName}`;
    response.data.pipe(fs.createWriteStream(localPath));

    // Process with Cline
    const content = fs.readFileSync(localPath, 'utf-8');
    const result = await clineService.analyzeCode(content, fileName);
    await bot.sendMessage(msg.chat.id, result, { parse_mode: 'Markdown' });
  });

8.2 Code Snippets
───────────────────────────────────────────────────────────────────────────────
Detect and handle code blocks sent in messages:

  bot.on('message', async (msg) => {
    const text = msg.text || '';
    // Detect markdown code blocks: ```language ... ```
    const codeBlockRegex = /```(\w+)?\n([\s\S]*?)```/g;
    let match;
    while ((match = codeBlockRegex.exec(text)) !== null) {
      const [, lang, code] = match;
      await clineService.analyzeCode(code.trim(), `snippet.${lang || 'txt'}`);
    }
  });

8.3 Sending Code Back
───────────────────────────────────────────────────────────────────────────────
Format responses with proper code blocks for Telegram:

  function formatCodeResponse(text) {
    // Escape Telegram markdown special characters
    return text
      .replace(/_/g, '\\_')
      .replace(/\*/g, '\\*')
      .replace(/\[/g, '\\[')
      .replace(/`/g, '\\`');
  }

  // Or send code as a file for larger outputs:
  bot.sendDocument(chatId, Buffer.from(code), {
    filename: 'output.py',
    caption: 'Generated code from Cline'
  });

    }
  });

6.5 /chat with Context
───────────────────────────────────────────────────────────────────────────────
  bot.onText(/\/chat (.+)/, async (msg, match) => {
    const message = match[1];
    const chatId = msg.chat.id;
    const userId = msg.from.id;

    const context = sessionManager.getContext(userId);
    context.push({ role: 'user', content: message });

    const response = await clineService.chat(context);
    context.push({ role: 'assistant', content: response });
    sessionManager.setContext(userId, context);

    await bot.sendMessage(chatId, response, { parse_mode: 'Markdown' });
  });

6.6 Non-Command Messages
───────────────────────────────────────────────────────────────────────────────
  bot.on('message', async (msg) => {
    if (msg.text && msg.text.startsWith('/')) return;
    const userId = msg.from.id;
    const mode = sessionManager.getDefaultMode(userId) || 'chat';
    // Route to /ask, /code, or /chat handler based on mode
  });

  5. Bridge sends request to Cline API.
  6. Cline processes and returns response.
  7. Bridge formats response for Telegram (markdown, code blocks).
  8. Bridge sends response back via Telegram Bot API.

5.3 Communication with Cline
───────────────────────────────────────────────────────────────────────────────
  Option A — Cline API:     POST http://localhost:PORT/api/chat
  Option B — Cline CLI:     cline "prompt" --mode code
  Option C — Direct LLM API: Call OpenAI/Anthropic API directly with
                             Cline-like system prompts (recommended).
  Option D — File-based:    Watch directory for input/output files.

5.4 Bridge Server Structure
───────────────────────────────────────────────────────────────────────────────
  cline-telegram-bridge/
  ├── src/
  │   ├── index.js          # Entry point, bot init
  │   ├── config.js         # Config & env vars
  │   ├── handlers/
  │   │   ├── commands.js   # /ask, /code, /chat
  │   │   ├── messages.js   # Free-form messages
  │   │   └── files.js      # File/document handling
  │   ├── services/
  │   │   ├── cline.js      # Cline API client
  │   │   ├── session.js    # Session management
  │   │   └── formatter.js  # Response formatting
  │   └── utils/
  │       ├── auth.js       # User authorization
  │       └── rate-limit.js # Rate limiting
  ├── .env
  ├── package.json
  └── README.md

  • Example .env file:
    TELEGRAM_BOT_TOKEN=7234567890:AAHdqTcvCH1vGWJxfSeofSAs0K5PALDsaw
    CLINE_API_URL=http://localhost:3000/api
    CLINE_API_KEY=sk-your-cline-key
    ALLOWED_USER_IDS=123456789,987654321

================================================================================
4. LIBRARY SELECTION
───────────────────────────────────────────────────────────────────────────────

4.1 Node.js — node-telegram-bot-api
───────────────────────────────────────────────────────────────────────────────
  Install: npm install node-telegram-bot-api

  Features:
    • Full Telegram Bot API coverage.
    • Built-in polling and webhook support.
    • File upload/download utilities.
    • Keyboard and inline query support.
    • TypeScript definitions included.

  Basic setup:
    const TelegramBot = require('node-telegram-bot-api');
    const bot = new TelegramBot(process.env.TELEGRAM_BOT_TOKEN, { polling: true });

    bot.onText(/\/start/, (msg) => {
      bot.sendMessage(msg.chat.id, "Hello! I'm your Cline bridge.");
    });

  3. Follow the prompts:
     - Choose a display name for your bot (e.g., "Cline Assistant").
     - Choose a username ending in "bot" (e.g., "ClineAssistantBot").
  4. BotFather will confirm creation and provide:
     - Bot token (e.g., 7234567890:AAHdqTcvCH1vGWJxfSeofSAs0K5PALDsaw)
     - Link to your bot (t.me/ClineAssistantBot)
     - Bot username

2.2 Configuring the Bot
───────────────────────────────────────────────────────────────────────────────
After creation, you can configure your bot with these BotFather commands:

  /setdescription   — Brief description shown on the bot profile.
  /setabouttext     — "About" text for the bot.
  /setuserpic       — Set a profile picture for the bot.
  /setcommands      — Define the command list menu (shown with /).
                      Example:
                      ask - Ask Cline a question
                      code - Execute a coding task
                      chat - Free-form chat with Cline
                      status - Check bot status
                      help - Show available commands
  /setprivacy       — Toggle privacy mode (recommended: Disabled for broad access).

2.3 Important BotFather Settings
───────────────────────────────────────────────────────────────────────────────
  • Privacy Mode: Enabled by default — bot only sees messages that start
    with a command or are directed at it. Disable for group chats.
  • Group Privacy: Set via /setprivacy. Disable if adding bot to groups.
  • Inline Mode: Allows invoking the bot by typing @username anywhere.

