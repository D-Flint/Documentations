# 📚 Documentations

A personal knowledge base documenting software configurations, fixes, and guides for various tools and systems I use. This repository serves as a reference to quickly recall solutions when things break — saving hours of searching the internet.

> **Disclaimer:** My only intention for this repository is to document what I do and how I do it, so that the next time something breaks, I don't have to scour the internet for hours to find the solution. If you find this information helpful, feel free to use it!

---

## 📋 Table of Contents

- [📚 Documentations](#-documentations)
  - [📋 Table of Contents](#-table-of-contents)
  - [🗂️ Repository Structure](#️-repository-structure)
  - [📂 Documentation Categories](#-documentation-categories)
    - [🐧 Arch Linux](#-arch-linux)
    - [🖥️ Fedora](#️-fedora)
    - [🤖 AI / Coding Assistants](#-ai--coding-assistants)
    - [🔧 Git](#-git)
    - [🌐 API Guides](#-api-guides)
  - [📖 How to Use](#-how-to-use)
  - [🤝 Contributing](#-contributing)

---

## 🗂️ Repository Structure

```
Documentations/
├── ARCH/                        # Arch Linux documentation
│   └── fixes/
│       └── keybind.md           # Keybinding fixes
├── Cline/
│   └── Cline.md                 # Cline AI coding assistant guide
├── cline-telegram-bot/
│   └── cline-telegram-bot.md    # Cline + Telegram Bot guide
├── deepseek-api/
│   └── deepseek-api.md          # DeepSeek API guide
├── Fedora/
│   └── command/
│       └── brightness-control.md # Brightness control for Fedora
├── git/
│   ├── git-basic-command.md     # Git basic commands reference
│   ├── git.md                   # Comprehensive Git guide
│   └── git-troubleshooting.md   # Git troubleshooting guide
└── README.md                    # This file
```

---

## 📂 Documentation Categories

### 🐧 Arch Linux

| Document | Description |
|----------|-------------|
| [Keybinding Fixes](ARCH/fixes/keybind.md) | Fixing brightness control keybindings in Arch Linux with GNOME, including installation of `brightnessctl`, clearing default GNOME keybinds, and setting up custom keybindings. |

### 🖥️ Fedora

| Document | Description |
|----------|-------------|
| [Brightness Control](Fedora/command/brightness-control.md) | Guide for controlling display brightness on Fedora using `brightnessctl`, including identifying display names and backlight drivers. |

### 🤖 AI / Coding Assistants

| Document | Description |
|----------|-------------|
| [Cline Guide](Cline/Cline.md) | Comprehensive guide to Cline (formerly Claude Dev) — an autonomous AI coding assistant for VS Code/Cursor/Windsurf. Covers installation, API provider setup, custom instructions, slash commands, checkpoints, MCP server integration, and best practices. |
| [Cline + Telegram Bot](cline-telegram-bot/cline-telegram-bot.md) | Guide for bridging Cline with Telegram via a bot. Covers BotFather setup, library selection (Node.js/Python), bridge architecture, command handling, session management, file handling, webhook vs polling, and deployment. |

### 🔧 Git

| Document | Description |
|----------|-------------|
| [Git Basic Commands](git/git-basic-command.md) | Quick reference for essential Git commands — init, clone, add, commit, push, pull, branch, merge, log, diff. |
| [Git Comprehensive Guide](git/git.md) | In-depth Git guide covering installation (Windows/Linux/macOS), configuration, branching strategies, merging vs rebasing, merge conflict resolution, stashing, tagging, .gitignore, remote management, PR workflows, bisect, reflog, and troubleshooting. |
| [Git Troubleshooting](git/git-troubleshooting.md) | Step-by-step solutions for common Git errors when pushing a local project to a new GitHub repository (e.g., `src refspec main does not match any`, merge conflicts on README, unrelated histories). |

### 🌐 API Guides

| Document | Description |
|----------|-------------|
| [DeepSeek API](deepseek-api/deepseek-api.md) | Detailed guide to the DeepSeek API — available models (DeepSeek-V2, DeepSeek-Coder-V2, DeepSeek-R1), authentication, endpoints, streaming, token limits, pricing, rate limits, integration with OpenRouter, and usage with Cline. |

---

## 📖 How to Use

Browse the directories above or use the table of contents to find the topic you need. Each document is a self-contained Markdown file with step-by-step instructions, commands, and explanations.

---

## 🤝 Contributing

This is a personal documentation repository, but if you spot an error or have a suggestion, feel free to open an issue or submit a pull request!