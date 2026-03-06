---
name: discord-auto-thread
description: Automatically create Discord threads for every new topic or question. Keeps main channels clean by routing all replies into threads. Use when the agent operates in Discord channels and should organize conversations into threads instead of replying directly in the channel. Triggers on any Discord channel message that starts a new topic, question, or discussion.
---

# Discord Auto-Thread

Keep Discord channels clean by automatically creating a thread for every new message that deserves a response.

## Core Rule

**Never reply directly in a Discord channel.** Always create a thread first, then reply inside it.

## ⚠️ Zero Narration Rule (CRITICAL)

When in a Discord channel (not a thread), **do not write ANY text between tool calls.** Any text outside a tool call gets sent directly to the channel, violating the auto-thread rule.

**Wrong:**
```
Let me look that up for you...     ← THIS LEAKS TO THE CHANNEL
[tool call: web_search]
[tool call: thread-create]
```

**Right:**
```
[tool call: web_search]            ← silent, no text
[tool call: read file]             ← silent, no text
[tool call: thread-create]         ← ALL response text goes here
```

Rule: **Channel context = zero text output. Only tool calls + final thread-create.**

## Workflow

When a message arrives in a Discord channel (not already in a thread):

1. Do any required work silently (search, read files, compute) — **no narration text**
2. Determine a short, descriptive thread name from the topic
3. Create a thread on the triggering message using `message(action=thread-create)`
4. Put your **complete** response in the `message` parameter of `thread-create`

## Follow-up Routing

After `thread-create`, the response includes a thread ID. All subsequent messages in that conversation **must** use `threadId` to reply inside the thread:

```
message(action=send, channel=discord, threadId=<thread_id>, message=...)
```

**Never send follow-ups back to the parent channel.** Once a thread exists, stay in it.

## Thread Naming

- Keep names short (3-8 words)
- Describe the topic, not the action (e.g. "Polymarket API setup" not "Answering question about Polymarket")
- Use the language of the original message

## Exceptions

- **Already in a thread** → reply directly, do not create a nested thread
- **Reactions only** → use `message(action=react)` directly, no thread needed
- **Simple acknowledgments** (👍, got it) → react instead of creating a thread

## Session Startup Check

At the start of every session, check:
1. Am I in a Discord channel or a thread?
2. If channel → every response goes through `thread-create`, no exceptions
3. If thread → reply normally

This check must happen **every session**. Knowing the rule is not enough — actively apply it from the first message.

## Example

User posts in #general: "有人知道布拉格哪裡有好吃的拉麵嗎？"

```
[tool call: web_search("Prague ramen")]     ← silent
[tool call: thread-create]                  ← all output here
    messageId: <triggering message id>
    threadName: "布拉格拉麵推薦"
    message: <your full answer>
```

Follow-up questions from anyone → automatically land in the thread.
