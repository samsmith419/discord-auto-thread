# discord-auto-thread

An [OpenClaw](https://openclaw.ai) skill that automatically creates Discord threads for every new topic, keeping your channels clean and organized.

## The Problem

AI agents reply directly in Discord channels. With multiple people asking questions, the main channel turns into an unreadable wall of interleaved conversations.

## The Fix

Install this skill and your agent will automatically create a thread on every new message, with the full reply inside the thread. The main channel stays clean.

## How It Works

| Situation | Agent behavior |
|-----------|---------------|
| New message in channel | Creates thread → replies inside |
| Already in a thread | Replies directly |
| Simple acknowledgment | Reacts with emoji, no thread |

- Thread names are auto-generated from the topic content (multilingual)
- Agent stays completely silent in the channel — no text leaks into main chat
- All follow-up messages stay inside the thread, never back to the parent channel
- Error handling built in: rate limits get retried, permission issues get surfaced, guild thread limits don't silently fail

## Before & After

**Before:** A noisy channel where you can't follow any conversation.

**After:** Clean channel with organized threads. Each topic self-contained. Like a built-in forum mode.

## Install

Clone and drop `SKILL.md` into your OpenClaw skills directory:

```bash
mkdir -p ~/.openclaw/skills/discord-auto-thread
curl -o ~/.openclaw/skills/discord-auto-thread/SKILL.md \
  https://raw.githubusercontent.com/samsmith419/discord-auto-thread/main/SKILL.md
```

Or clone the whole repo:

```bash
git clone https://github.com/samsmith419/discord-auto-thread.git ~/.openclaw/skills/discord-auto-thread
```

One file. No scripts. No dependencies.

## License

MIT
