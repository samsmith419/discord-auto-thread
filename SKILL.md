---
name: discord-auto-thread
description: Automatically create Discord threads for every new topic or question. Keeps main channels clean by routing all replies into threads. Use when the agent operates in Discord channels and should organize conversations into threads instead of replying directly in the channel. Triggers on any Discord channel message that starts a new topic, question, or discussion.
---

# Discord Auto-Thread

Keep Discord channels clean by automatically creating a thread for every new message that deserves a response.

## Core Rule

**Never reply directly in a Discord channel.** Always create a thread first, then reply inside it.

## Workflow

When a message arrives in a Discord channel (not already in a thread):

1. Determine a short, descriptive thread name from the topic (e.g. "Prague weather this weekend", "Git rebase vs merge")
2. Create a thread on the triggering message using `message(action=thread-create)`
3. Put your full response in the `message` parameter of `thread-create`
4. All follow-up discussion continues inside the thread automatically

## Thread Naming

- Keep names short (3-8 words)
- Describe the topic, not the action (e.g. "Polymarket API setup" not "Answering question about Polymarket")
- Use the language of the original message

## Exceptions

- **Already in a thread** → reply directly in the thread, do not create a nested thread
- **Reactions only** → use `message(action=react)` directly, no thread needed
- **Simple acknowledgments** (👍, got it) → react instead of creating a thread

## Example

User posts in #general: "有人知道布拉格哪裡有好吃的拉麵嗎？"

```
→ message(action=thread-create)
    messageId: <triggering message id>
    threadName: "布拉格拉麵推薦"
    message: <your full answer>
```

Follow-up questions from anyone → automatically land in the thread.
