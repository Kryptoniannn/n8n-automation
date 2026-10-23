<div align="center">

<img src="https://capsule-render.vercel.app/api?type=waving&color=0:0B1220,55:EA4B71,100:FFD93F&height=160&section=header&text=n8n%20Automation%20Suite&fontSize=40&fontColor=EAF2FF&animation=fadeIn&fontAlignY=38&desc=AI%20agents%20wired%20to%20the%20real%20world&descAlignY=60&descSize=15" width="100%" />

**Importable, sanitized n8n workflows — support automation, AI video production, and prompt iteration.**

<br/>

<img src="https://img.shields.io/badge/n8n-EA4B71?style=flat-square&logo=n8n&logoColor=white" />
<img src="https://img.shields.io/badge/OpenAI-412991?style=flat-square&logo=openai&logoColor=white" />
<img src="https://img.shields.io/badge/Gmail-EA4335?style=flat-square&logo=gmail&logoColor=white" />
<img src="https://img.shields.io/badge/Kie.ai%20Veo-FFD93F?style=flat-square&logoColor=0B1220" />
<img src="https://img.shields.io/badge/License-MIT-2FD5A0?style=flat-square" />

</div>

---

## What's inside

| # | Workflow | Trigger | What it does |
|---|---|---|---|
| 01 | **Gmail AI Support Agent** | Gmail (poll) | Reads incoming mail, runs an LLM agent with a business policy prompt, replies in-thread |
| 02 | **Viral Video Factory** | Form webhook | Idea → detailed video prompt → Veo 3 generation → poll for result → auto-publish |
| 03 | **Agent Playground** | Manual | Minimal sandbox for iterating on a prompt and model before wiring it into production |

Every file is a **ready-to-import n8n workflow**. Credentials and keys have been stripped — connect your own after import.

---

## 01 · Gmail AI Support Agent

```
Gmail Trigger ──► AI Agent (system prompt = business policy) ──► Gmail reply
                        ▲
                   Chat Model
```

The interesting part is not the plumbing — it's the policy. The system prompt encodes hard business rules (delivery area, refund window, tone) so the model refuses to improvise:

```
We ONLY deliver within Dhaka. If asked about delivery to any other city,
clearly say we currently deliver to Dhaka only.
Refunds only for size mismatch, requested within 3 days of delivery.
Reply politely and concisely, like a real support agent.
```

Replies go back as `Re: <original subject>` to the original sender, with n8n's attribution removed — so it reads like a human wrote it.

**Setup:** connect a Gmail OAuth2 credential, an OpenAI credential, then edit the system prompt to your own business rules.

---

## 02 · Viral Video Factory

```
Form ──► AI Agent (video-prompt writer) ──► title/hashtag model
                                              │
                                              ▼
                                        POST /veo/generate
                                              │
                                        Wait 200s
                                              │
                                        GET /veo/record-info ──► upload-post
```

A prompt-engineering pipeline rather than an API demo. The agent is instructed to produce production-ready "found footage" style prompts: Bangla dialogue, local setting, camera imperfections, aspect ratio, timestamp overlays — the details that make generated video feel real rather than synthetic.

The generation model runs asynchronously, so the flow waits and then polls `record-info` by `taskId` before publishing.

**Note on the form trigger:** it accepts both `GET` and `POST`, so the form URL is directly shareable as a hosted input page.

---

## 03 · Agent Playground

Manual trigger → one agent node → one model. Deliberately tiny. It exists so prompt changes can be tested in isolation instead of against a live inbox.

---

## Import

1. In n8n: **Workflows → Import from File**
2. Pick a JSON from `workflows/`
3. Reconnect credentials (each node that needs one is left unlinked on purpose)
4. Update the env vars below, then activate

### Environment variables

| Variable | Used by | Purpose |
|---|---|---|
| `KIE_API_KEY` | 02 | Kie.ai bearer token for Veo generation |
| `KIE_CALLBACK_URL` | 02 | Optional completion callback |

API keys are referenced as `{{ $env.KIE_API_KEY }}` rather than pasted into nodes, so the workflow JSON stays safe to share.

---

## Design notes

- **No secrets in the repo.** Exports are sanitized before publishing; keys live in env vars.
- **Policy in the prompt, not in code.** Business rules belong where a non-engineer can edit them.
- **Async by default.** Video generation takes minutes; the flow waits and polls instead of pretending it's instant.
- **Small nodes, single responsibility.** Each step is debuggable on its own.

---

## Roadmap

- Error-handling branch with retry + alert on failed generation
- Vector-store retrieval for the support agent instead of a static prompt
- Structured logging of token cost per conversation

---

**Built by [MD Shahidul Islam](https://www.linkedin.com/in/md-shahidul-islam-10910a375)** — AI automation and full-stack engineering, Bangladesh.

