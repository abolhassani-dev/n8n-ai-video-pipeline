# AI Video Pipeline — n8n + Telegram Control

> Multi-stage automation workflow generating short-form video from a single Telegram command — orchestrating GPT-4, Fal.ai Flux, Kling.ai, ElevenLabs, and Creatomate.

A production-grade [n8n](https://n8n.io) workflow that takes a topic from a Telegram message and returns a finished short-form video ready for review — all through a single `/generate` command.

This is the workflow architecture I built and ran as a personal R&D project to explore multi-API AI orchestration. It demonstrates real production concerns: async polling, error handling, status checks, and human-in-the-loop approval.

---

## Pipeline

```
Telegram /generate command
         │
         ▼
GPT-4 (Generate prompts: image + video motion + voiceover + title)
         │
         ▼
Fal.ai Flux (Image generation, async queue)
         │
         ▼  (poll until COMPLETED)
Kling.ai (Image → Video animation)
         │
         ▼  (poll until succeed)
ElevenLabs (Text-to-speech voiceover)
         │
         ▼
Creatomate (Final video render with title + audio overlay)
         │
         ▼  (poll until rendered)
Telegram (Send back to admin for approval)
```

Each stage has its own polling logic because the APIs are asynchronous — Fal.ai and Kling can take 15–60 seconds, Creatomate even longer. The workflow handles all of that without blocking.

---

## What it demonstrates

- **Multi-API orchestration** — chaining 5 third-party services with different auth, response formats, and async patterns
- **Async polling** — handling queue endpoints with status checks (Fal.ai, Kling, Creatomate)
- **Error handling** — pipeline failure detection at every stage with Telegram notifications back to the user
- **Human-in-the-loop** — the final result goes back to Telegram for review before any publishing
- **Stateful context across stages** — using n8n expressions like `$('Node Name').item.json` to thread data through the pipeline
- **Production-ready credentials** — separated credential references for each external service

---

## File

[`ai-video-pipeline.json`](ai-video-pipeline.json) — full workflow ready to import into n8n.

To use:
1. Import the JSON into your n8n instance (`Workflows → Import from File`).
2. Configure credentials for: Telegram Bot, OpenAI, Fal.ai, Kling.ai, ElevenLabs, Creatomate.
3. Replace `YOUR_CREATOMATE_TEMPLATE_ID` in the Creatomate node with your own template.
4. Activate the workflow.
5. Send `/generate <your topic>` to the Telegram bot.

---

## Notes on production hardening

This workflow as shared is the **architecture skeleton**. In the live version I ran, additional layers were added on top:

- **Rate limiting** per Telegram user (so a single user can't blast the GPT-4 quota)
- **Retry logic** with exponential backoff on API failures
- **Cost tracking** — each pipeline run logs token/credit usage per provider
- **Approval queue** — `/approve` and `/reject` commands trigger separate sub-workflows for publishing or discarding
- **Stuck-job detection** — a parallel monitor workflow cleans up runs that exceed maximum allowed time

Those were specific to my deployment environment and are not included here, but the architecture supports them naturally.

---

## Why I built this

I wanted to see how far automated content generation could go before quality breaks down. The answer is: surprisingly far, but human review remains essential. The Telegram-based control layer is what made it usable — being able to trigger and review runs from anywhere without opening a browser turned this from a script into something I'd actually run.

---

## Tech stack

| Layer | Tool |
|-------|------|
| Orchestration | n8n (self-hosted on Linux VPS) |
| LLM | OpenAI GPT-4 Turbo |
| Image generation | Fal.ai Flux Pro |
| Video animation | Kling.ai v1 |
| Voiceover | ElevenLabs (multilingual v2) |
| Final render | Creatomate |
| Control surface | Telegram Bot API |

---

## Author

**Hamidreza Abolhassani**
Email: hr.dev.abolhassani@gmail.com
