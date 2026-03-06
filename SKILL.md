---
name: discord-auto-thread
description: Automatically create Discord threads for every new topic or question. Keeps main channels clean by routing all replies into threads. Use when the agent operates in Discord channels and should organize conversations into threads instead of replying directly in the channel. Triggers on any Discord channel message that starts a new topic, question, or discussion.
---

# Discord Auto-Thread

Keep Discord channels clean by automatically creating a thread for every new message that deserves a response.

## Core Rules

1. **Never reply directly in a Discord channel.** Always create a thread first.
2. **Your full response goes in the `message` parameter of `thread-create`.** Do NOT send a separate `message(action=send)` after creating the thread.
3. **After a thread exists, all follow-up messages stay in the thread.** Use `threadId` for any subsequent sends — never send back to the parent channel.

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
[tool call: thread-create]         ← ALL response text goes here
```

Rule: **Channel context = zero text output. Only tool calls + final thread-create.**

## Workflow

When a message arrives in a Discord channel (not already in a thread):

1. Do any required work silently (search, read files, compute) — **no narration text**
2. Determine a short thread name from the topic (3–8 words, max 100 chars)
3. Call `message(action=thread-create)` with:
   - `messageId`: the triggering message id
   - `threadName`: short topic description
   - `message`: your **complete** response
4. Reply `NO_REPLY`

Do NOT call `message(action=send)` separately — `thread-create` already delivers your response inside the thread.

## After Thread Creation

**Never send messages back to the parent channel after opening a thread.**

Follow-up messages in the same conversation:
```
message(action=send, channel=discord, threadId=<thread_id>, message=...)
```
The thread ID is returned in the `thread-create` response (`thread.id`).

If the conversation continues (user replies in thread), you're already in thread context — reply normally.

## Thread Naming

- 3–8 words, max 100 characters
- Describe the topic, not the action ("Polymarket API setup" not "Answering question about Polymarket")
- Use the language of the original message

## Exceptions

- **Already in a thread** → reply directly, do not create nested threads
- **Reactions only** → use `message(action=react)` directly, no thread needed
- **Simple acknowledgments** (👍, got it) → react instead of creating a thread
- **thread-create fails with rate limit error** → wait and retry once; if it fails again, send a brief apology message to the channel explaining the delay (this is the only acceptable reason to send directly to a channel)
- **thread-create fails because guild hit active thread limit (~1000)** → reply in the channel and note the limitation; do not silently drop the response

## Discord Thread Constraints (Know These)

| Constraint | Detail |
|---|---|
| Active thread limit | ~1000 per guild; near limit → threads auto-archive faster |
| Auto-archive | Threads archive after inactivity (default 1–7 days depending on guild boost level) |
| Sending unarchives | Sending a message to an archived thread automatically unarchives it (unless locked) |
| Locked threads | Only MANAGE_THREADS permission can send to or unarchive a locked thread |
| Thread name max | 100 characters |
| Permissions | `SEND_MESSAGES` ≠ `SEND_MESSAGES_IN_THREADS` — these are separate permission bits |
| API version | Thread events require Discord API v9+ |

## Session Startup Check

At the start of every session, check:
1. Am I in a Discord channel or a thread?
2. If channel → every response goes through `thread-create`, no exceptions
3. If thread → reply normally

This check must happen **every session**. Knowing the rule is not enough — actively apply it from the first message.

## Example

User posts in #general: "有人知道布拉格哪裡有好吃的拉麵嗎？"

```
[tool call: web_search("Prague ramen")]   ← silent
[tool call: thread-create]                ← all output here
    messageId: <triggering message id>
    threadName: "布拉格拉麵推薦"
    message: "推薦幾家：\n- ..."
→ reply NO_REPLY
```

**❌ Wrong — sending separately goes to main channel:**
```
→ message(action=thread-create, messageId=..., threadName=...)
→ message(action=send, target=channel:xxx, message="推薦幾家...")  ← WRONG
```

**✅ Follow-up after thread-create:**
```
→ message(action=send, threadId=<thread_id>, message="補充一下...")  ← stays in thread
```

## Error Recovery

If `thread-create` fails:
1. Check the error: rate limit → wait/retry; permission denied → log and skip; guild thread limit → notify user in channel
2. Never silently swallow errors — always provide a response via the safest available path
3. If the original message is deleted before thread creation, `thread-create` will fail — treat as a no-op
