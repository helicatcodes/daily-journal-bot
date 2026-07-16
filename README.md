# Daily AI Journal Bot

A Telegram-based journaling companion that turns daily reflection into a lasting habit. Every morning at 7am it runs a guided five-prompt check-in — mood, gratitude, dreams, micro-victories, intention — accepting text or voice notes, then generates a structured summary, a personalised affirmation, and a unique AI image, all archived automatically to Notion.

My first n8n project. Built in a couple of evenings, running every morning since.

🎬 [Demo](https://www.youtube.com/watch?v=ZIpYAGTFZqM) · [Full execution run](https://youtu.be/TxRlTwfj3TM)

<img width="3357" height="1439" alt="Daily Journal Bot Demo Flow" src="https://github.com/user-attachments/assets/374fbaba-370a-4faf-8994-723c9acfac14" />


## The engineering challenge

n8n workflows run as single executions — they can't natively pause and wait for a human reply, which makes true multi-turn conversation impossible out of the box.

I solved this with a **stateful conversation loop**: after each question, the workflow pauses on a webhook and stores its resume URL in a lookup table keyed to the chat. When the next Telegram message arrives, a lightweight receiver workflow looks up the stored URL and resumes the paused execution exactly where it left off. The result is a genuine back-and-forth dialogue on a platform that doesn't support one.

## How it works

1. **Scheduled trigger** — a cron node kicks off the check-in at 7am via Telegram, so the ritual starts with zero effort.
2. **Guided AI conversation** — a GPT-4 agent with conversation memory asks the five prompts one at a time, keeping a warm, encouraging tone.
3. **Voice & text input** — an IF-node routing layer detects voice notes, transcribes them with Whisper, and normalises both branches in a Code node so the agent always receives clean text.
4. **Summary & affirmation** — on completion, the agent produces a structured, emoji-labelled summary plus a personalised affirmation (second person, present tense).
5. **Daily AI image** — a parallel pipeline builds an image prompt from the day's intention, calls OpenAI's image API, decodes the base64 response, and delivers the art to Telegram.
6. **Notion archive** — regex-based field extraction parses the structured summary into a typed Notion database, building a searchable long-term journal.

The design is rooted in behavioural psychology: minimal friction, a well-timed cue, and a familiar interface (a chat app already on my home screen).

## Tech stack

- **n8n** — workflow orchestration, AI agent hosting, conversational loop architecture, scheduling
- **OpenAI** — GPT-4 (journal agent), Whisper (voice transcription), gpt-image-1 (daily image)
- **Telegram Bot API** — conversational interface and delivery
- **Notion API** — structured journal database
- **JavaScript (n8n Code nodes)** — input normalisation, base64 decoding, data formatting

## Repo contents

- `workflow.json` — the exported n8n workflow (credentials stripped; IDs replaced with placeholders like `YOUR_NOTION_DB_ID`)
- `docs/` — screenshots of the workflow, Telegram flow, and Notion archive

To run it yourself: import `workflow.json` into n8n, connect your own Telegram bot, OpenAI, and Notion credentials, and replace the placeholder IDs.

## Key learnings

- **Stateful workflow design** — webhook resume URLs + a lookup table enable conversational flows n8n doesn't support natively
- **Structured output as an integration bridge** — one AI output format serving both human readability (Telegram) and machine parsing (Notion)
- **Multi-modal input handling** — conditional routing with graceful fallbacks for voice vs. text
- **Pragmatic scoping** — core conversation first, then image generation and error handling
