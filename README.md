# NoxGram

> Telegram Bot framework primitives for the [Nox](https://github.com/devnexe/nox) programming language.

[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Nox](https://img.shields.io/badge/nox-0.1.0%2B-purple.svg)](https://github.com/devnexe/nox)
[![Telegram Bot API](https://img.shields.io/badge/Telegram%20Bot%20API-latest-26A5E4.svg)](https://core.telegram.org/bots/api)

---

## What is NoxGram?

**NoxGram** is a lightweight Telegram bot framework built natively for Nox. It gives you command routing, message filtering, FSM-based state management, and a long-polling loop — all in pure Nox, with no external dependencies beyond the standard `http` module.

```nox
import NoxGram as tg

bot = tg.bot("YOUR_TOKEN")

bot.command("start", define(ctx):
    ctx["bot"].send_message(ctx["chat_id"], "Hello from Nox! 👋")
)

await bot.poll()
```

---

## Installation

```bash
nox package install NoxGram
```

**Requirements:**
- Nox `0.1.0+`
- Standard `http` module (built into Nox)

---

## Quick Start

### Echo Bot

```nox
import NoxGram as tg

bot = tg.bot("YOUR_BOT_TOKEN")

bot.on_message(define(ctx):
    text = ctx["text"]
    if text != none:
        ctx["bot"].send_message(ctx["chat_id"], "You said: " + text)
)

await bot.poll()
```

### Commands

```nox
bot.command("help", define(ctx):
    ctx["bot"].send_message(ctx["chat_id"], "Available commands: /start /help")
)

bot.command("start", define(ctx):
    ctx["bot"].send_message(ctx["chat_id"], "Welcome!")
)
```

### FSM (Finite State Machine)

NoxGram has a built-in FSM for tracking per-user conversation state:

```nox
bot.command("setname", define(ctx):
    bot.state_set(ctx["chat_id"], "waiting_name")
    ctx["bot"].send_message(ctx["chat_id"], "What's your name?")
)

bot.on_message(define(ctx):
    if ctx["state"] == "waiting_name":
        bot.state_clear(ctx["chat_id"])
        ctx["bot"].send_message(ctx["chat_id"], "Nice to meet you, " + ctx["text"] + "!")
)
```

### Custom Filters

```nox
only_hello = tg.filter_text_contains("hello")

bot.on_filter(only_hello, define(ctx):
    ctx["bot"].send_message(ctx["chat_id"], "Hey there! 👋")
)
```

---

## API Reference

### `bot(token)`

Creates and returns a new `Bot` instance.

```nox
bot = tg.bot("TOKEN")
```

---

### `Bot`

#### Handlers

| Method | Description |
|--------|-------------|
| `bot.command(name, handler)` | Register a handler for `/name` commands |
| `bot.on_message(handler)` | Register a handler for all incoming messages |
| `bot.on_filter(filter_fn, handler)` | Register a handler that runs when `filter_fn(ctx)` returns `true` |
| `bot.use(middleware)` | Register a middleware that runs before every dispatch |

#### State (FSM)

| Method | Description |
|--------|-------------|
| `bot.state_set(chat_id, state)` | Set the FSM state for a chat |
| `bot.state_get(chat_id)` | Get the current FSM state for a chat |
| `bot.state_clear(chat_id)` | Clear the FSM state for a chat |

#### Sending

| Method | Description |
|--------|-------------|
| `bot.send_message(chat_id, text)` | Send a plain-text message to a chat |

#### Polling

```nox
await bot.poll(interval_ms=700, timeout=20, max_batch=100)
```

| Parameter | Default | Description |
|-----------|---------|-------------|
| `interval_ms` | `700` | Milliseconds to sleep between empty polling cycles |
| `timeout` | `20` | Long-poll timeout in seconds sent to Telegram |
| `max_batch` | `100` | Maximum number of updates to process per cycle |

---

### `FSM`

Standalone finite state machine (also used internally by `Bot`).

```nox
fsm = tg.FSM()
fsm.set("user_123", "step_1")
state = fsm.get("user_123")  # "step_1"
fsm.clear("user_123")
```

---

### Filters

Built-in filter factories for use with `bot.on_filter`:

| Filter | Description |
|--------|-------------|
| `filter_command(name)` | Matches messages that are the `/name` command |
| `filter_text_contains(part)` | Matches messages that start with `part` |

Custom filters are just functions that take a `ctx` and return `true` or `false`:

```nox
define my_filter(ctx):
    result ctx["text"] == "ping"

bot.on_filter(my_filter, define(ctx):
    ctx["bot"].send_message(ctx["chat_id"], "pong")
)
```

---

### Context Object (`ctx`)

Every handler receives a `ctx` dictionary with the following keys:

| Key | Type | Description |
|-----|------|-------------|
| `bot` | `Bot` | The bot instance |
| `update` | `dict` | Raw Telegram update object |
| `message` | `dict` | The `message` field of the update |
| `text` | `str \| none` | Text of the message, or `none` |
| `chat_id` | `int \| none` | Chat ID of the sender |
| `state` | `any` | Current FSM state for this chat |

---

## Middleware

Middlewares run before every dispatch and can be used for logging, auth, etc.:

```nox
define logger(ctx):
    if ctx["text"] != none:
        print("[LOG] " + string(ctx["chat_id"]) + ": " + ctx["text"])

bot.use(logger)
```

---

## Full Example: Simple Quiz Bot

```nox
import NoxGram as tg

bot = tg.bot("YOUR_TOKEN")

QUESTION = "What is 2 + 2?"

bot.command("quiz", define(ctx):
    bot.state_set(ctx["chat_id"], "quiz_answer")
    ctx["bot"].send_message(ctx["chat_id"], QUESTION)
)

bot.on_message(define(ctx):
    if ctx["state"] == "quiz_answer":
        bot.state_clear(ctx["chat_id"])
        if ctx["text"] == "4":
            ctx["bot"].send_message(ctx["chat_id"], "✅ Correct!")
        else:
            ctx["bot"].send_message(ctx["chat_id"], "❌ Wrong! The answer is 4.")
)

await bot.poll()
```

---

## Project Layout

```
my-bot/
├── .nxinfo
├── main.nox          # Your bot entry point
└── handlers/
    ├── start.nox
    └── quiz.nox
```

---

## Notes

- Bot messages are automatically ignored (no infinite loops).
- `poll()` is async — call it with `await` from your top-level async context.
- FSM state is in-memory and resets on restart. For persistence, save/load state via `fs` or a database.

---

## License

MIT © DevNexe
