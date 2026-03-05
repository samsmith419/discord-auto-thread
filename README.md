# discord-auto-thread

An [OpenClaw](https://openclaw.ai) skill that automatically creates Discord threads for every new topic or question, keeping your channels clean and organized.

## What it does

When your agent receives a message in a Discord channel, instead of replying directly in the channel, it:

1. Creates a thread on the original message with a short, descriptive name
2. Posts the full reply inside the thread
3. All follow-up discussion stays contained in the thread

## Install

```bash
openclaw skill install samsmith419/discord-auto-thread
```

Or download `discord-auto-thread.skill` from [Releases](https://github.com/samsmith419/discord-auto-thread/releases).

## Behavior

| Situation | Action |
|-----------|--------|
| New message in channel | Create thread → reply inside |
| Already in a thread | Reply directly |
| Simple acknowledgment | React with emoji, no thread |

## License

MIT
