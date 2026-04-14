# Claude Code Telegram Channel Plugin (with Channel Support)

Connect a Telegram bot to your Claude Code with full support for **DM, Groups, and Channels**.

This is a fork of the official `telegram@claude-plugins-official` plugin (v0.0.4), with the following bugs fixed and features added.

---

## What's New vs Official v0.0.4

### Bug Fixes

| # | Location | Issue | Fix |
|---|----------|-------|-----|
| 1 | `readAccessFile()` | `channels` field not read from `access.json` — always `undefined` | Added `channels: parsed.channels ?? {}` |
| 2 | `assertAllowedChat()` | No check for `channels` allowlist — bot refused to reply to channels | Added `if (chat_id in access.channels) return` |
| 3 | `gate()` | No `channel` type branch — channel_post messages always dropped (no `from` field causes crash) | Added channel branch before the `from` guard |
| 4 | `handleInbound()` | `ctx.from!` crashes on channel_post (no sender) | Changed to `ctx.from ?? { id: Number(ctx.chat!.id), ... }` |
| 5 | `handleInbound()` | `msgId` only read from `ctx.message` — undefined for channel_post | Changed to `(ctx.message ?? ctx.channelPost)?.message_id` |
| 6 | `handleInbound()` | `ts` only read from `ctx.message?.date` — wrong for channel_post | Changed to `(ctx.message ?? ctx.channelPost)?.date` |

### New Features

| # | Feature | Detail |
|---|---------|--------|
| 7 | **Channel message reception** | Added 7 `channel_post:*` handlers: `text`, `photo`, `document`, `voice`, `audio`, `video`, `sticker` |
| 8 | **Terminal visibility** | Added `process.stderr.write` in `handleInbound` — channel messages now print to the Claude Code terminal |
| 9 | **`allowed_updates` in `bot.start`** | Added explicit `allowed_updates` list including `channel_post` and `edited_channel_post` so Telegram delivers channel updates |

---

## Prerequisites

- [Bun](https://bun.sh) — the MCP server runs on Bun
  ```sh
  curl -fsSL https://bun.sh/install | bash
  ```

---

## Quick Setup

### 1. Create a bot with BotFather

Open [@BotFather](https://t.me/BotFather) on Telegram and send `/newbot`. You'll receive a token like `123456789:AAHfiqksKZ8...`.

### 2. Install this plugin

In a Claude Code session:

```sh
/plugin install https://github.com/<your-username>/claude-telegram-channel-plugin
```

Or point to the local directory if you cloned it:

```sh
/plugin install /path/to/claude-telegram-channel-plugin
```

### 3. Save the bot token

```
/telegram:configure 123456789:AAHfiqksKZ8...
```

### 4. Relaunch with the channel flag

```sh
claude --channels plugin:telegram@claude-plugins-official
```

> If installed from this repo, replace the plugin identifier with the one shown after `/plugin install`.

### 5. Add your Telegram Channel to the bot

In `~/.claude/channels/telegram/access.json`, add your channel ID under `channels`:

```json
{
  "dmPolicy": "allowlist",
  "allowFrom": ["<your-user-id>"],
  "groups": {},
  "channels": {
    "-1001234567890": {
      "requireMention": false,
      "allowFrom": []
    }
  },
  "pending": {}
}
```

Make your bot an **admin** of the Telegram channel (required for the bot to post replies back).

### 6. Pair your DM (optional, for personal use)

DM your bot — it replies with a 6-character code. In Claude Code:

```
/telegram:access pair <code>
```

---

## Access Control

See **[ACCESS.md](./ACCESS.md)** for full schema, group support, mention detection, and delivery config.

---

## Tools Exposed to the Assistant

| Tool | Purpose |
|------|---------|
| `reply` | Send a message to a chat (DM, group, or channel). Supports `reply_to`, `files` (images/docs), and `markdownv2` format. Auto-chunks long text. |
| `react` | Add an emoji reaction (Telegram whitelist only: 👍 👎 ❤ 🔥 👀 🎉 etc). |
| `edit_message` | Edit a previously sent bot message. |
| `download_attachment` | Download an inbound file attachment to local inbox. |

---

## How Channel Messages Work

1. Telegram delivers channel posts as `channel_post` updates (not `message`)
2. The bot must be a **channel admin** to receive and reply
3. The channel ID must be listed in `access.json` under `channels`
4. Claude Code receives the message via MCP `notifications/claude/channel`
5. Claude replies using `chat_id` of the channel — reply appears in the channel

---

## Preventing Cache Overwrites

The official plugin cache at `~/.claude/plugins/cache/claude-plugins-official/telegram/0.0.4/` **will be overwritten** when the official plugin updates. To protect your fixes:

- **Use this repo** as your plugin source — install via GitHub URL, not the official marketplace
- The plugin installed from this repo lives in its own cache path, separate from the official one
- Do **not** rely on editing files under `~/.claude/plugins/cache/claude-plugins-official/`

---

## License

Apache-2.0 — same as the upstream official plugin.
