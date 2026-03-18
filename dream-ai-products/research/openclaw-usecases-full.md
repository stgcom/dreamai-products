# OpenClaw Use Cases — Full Collection

Source: github.com/hesamsheikh/awesome-openclaw-usecases
Plus community sources: useclaw.vercel.app, Robo Nuggets, forwardfuture.ai

### OpenClaw as Desktop Cowork (AionUi) — Remote Rescue & Multi-Agent Hub
File: aionui-cowork-desktop
# OpenClaw as Desktop Cowork (AionUi) — Remote Rescue & Multi-Agent Hub

Use OpenClaw from a desktop Cowork UI, access it from Telegram or WebUI when you’re away, and fix it remotely when it won’t connect. AionUi is a free, open-source app that runs **OpenClaw as a first-class agent** alongside 12+ others (Claude Code, Codex, Qwen Code, etc.), with a built-in **OpenClaw deployment expert** for install, diagnose, and repair — including **remote rescue** when OpenClaw is down and you’re not at the machine.

## Why OpenClaw + AionUi

| If you want… | AionUi gives you… |
|---------------|--------------------|
| **Use OpenClaw with a real desktop UI** | Cowork workspace where you see OpenClaw (and other agents) read/write files, run commands, browse the web — not just terminal/chat. |
| **Fix OpenClaw when it’s broken and you’re remote** | Open AionUi via **Telegram or WebUI** from anywhere → use the **OpenClaw deployment expert** to run `openclaw doctor`, fix config, restart gateway. Many users rely on this. |
| **One place for OpenClaw + other agents** | OpenClaw, built-in agent, Claude Code, Codex, etc. in one app; switch or run in parallel, same MCP config for all. |
| **Remote access to your OpenClaw** | WebUI, Telegram, Lark, DingTalk — talk to the same AionUi instance (and thus OpenClaw) from phone or another device. |

## Pain Point

You already use OpenClaw from CLI or Telegram, but:

- You want to **see** what the agent is doing (files, terminal, web) instead of inferring from logs.
- When **OpenClaw won’t connect** and you’re not at the machine, you have no way to run `openclaw doctor` or fix config — you need remote access to something that can repair OpenClaw.
- You use several CLI agents (OpenClaw, Claude Code, Codex, …) and don’t want to juggle apps or reconfigure MCP for each.
-e 
---

### arXiv Paper Reader
File: arxiv-paper-reader
# arXiv Paper Reader

Reading arXiv papers means downloading PDFs, losing context when switching between papers, and struggling to parse dense LaTeX notation. You want to read, analyze, and compare papers conversationally without leaving your workspace.

This workflow turns your agent into a research reading assistant:

- Fetch any arXiv paper by ID and get clean, readable text (LaTeX flattened automatically)
- Browse paper structure first — list sections to decide what to read before committing to the full text
- Quick-scan abstracts across multiple papers to triage a reading list
- Ask the agent to summarize, compare, or critique specific sections
- Results are cached locally — revisiting a paper is instant

## Skills you Need

- [arxiv-reader](https://github.com/Prismer-AI/Prismer/tree/main/skills/arxiv-reader) skill (3 tools: `arxiv_fetch`, `arxiv_sections`, `arxiv_abstract`)

No Docker or Python required — the skill runs standalone using Node.js built-ins. It downloads directly from arXiv, decompresses the LaTeX source, and flattens includes automatically.

## How to Set it Up

-e 
---

### Autonomous Educational Game Development Pipeline
## Pain Point
File: autonomous-game-dev-pipeline
# Autonomous Educational Game Development Pipeline

## Pain Point
**The Origin Story:** A "LANero of the old school" dad wanted to create a safe, ad-free, and high-quality gaming portal for his daughters, Susana (3) and Julieta (coming soon). Existing sites were plagued with spam, aggressive ads, and deceptive buttons (dark patterns) that frustrated his toddler.

**The Challenge:** Building a "clean, fast, and simple" portal was the easy part. The real challenge was populating it with **40+ educational games** tailored to specific developmental stages (0-15 years) without a team of developers. Manual development was too slow for a solo parent-developer, and maintaining consistency across dozens of games was becoming a nightmare.

## What It Does
This use case defines a "Game Developer Agent" that autonomously manages the entire lifecycle of a game's creation and maintenance. The workflow enforces a **"Bugs First"** policy where the agent must check for and resolve reported bugs before implementing new features.

**Efficiency:** This pipeline is capable of producing **1 new game or bugfix every 7 minutes**. The agent tirelessly iterates through the backlog of 41+ planned games, alternating between creating new content and correcting issues detected in previous cycles.

When the path is clear, the agent:
1.  **Selects**: Identifies the next game from a queue (`development-queue.md`) based on a "Round Robin" strategy to balance content across age groups.
2.  **Implements**: Writes HTML5/CSS3/JS code for the game, following strict `game-design-rules.md` (no frameworks, mobile-first, offline-capable).
3.  **Registers**: Automatically adds the game metadata to the central registry (`games-list.json`).
4.  **Documents**: Updates the `CHANGELOG.md` and `master-game-plan.md` status.
5.  **Deploys**: Handles the Git workflow: fetching master, creating a feature branch, committing changes with conventional commits, and merging back.

## Prompts
-e 
---

### Autonomous Project Management with Subagents
File: autonomous-project-management
# Autonomous Project Management with Subagents

Managing complex projects with multiple parallel workstreams is exhausting. You end up context-switching constantly, tracking status across tools, and manually coordinating handoffs.

This use case implements a decentralized project management pattern where subagents work autonomously on tasks, coordinating through shared state files rather than a central orchestrator.

## Pain Point

Traditional orchestrator patterns create bottlenecks—the main agent becomes a traffic cop. For complex projects (multi-repo refactors, research sprints, content pipelines), you need agents that can work in parallel without constant supervision.

## What It Does

- **Decentralized coordination**: Agents read/write to a shared `STATE.yaml` file
- **Parallel execution**: Multiple subagents work on independent tasks simultaneously
- **No orchestrator overhead**: Main session stays thin (CEO pattern—strategy only)
- **Self-documenting**: All task state persists in version-controlled files

## Core Pattern: STATE.yaml

Each project maintains a `STATE.yaml` file that serves as the single source of truth:
-e 
---

### Multi-Agent Content Factory
File: content-factory
# Multi-Agent Content Factory

You're a content creator juggling research, writing, and design across multiple platforms. Each step — finding trending topics, writing scripts, generating thumbnails — eats hours of your day. What if a team of specialized agents handled all of it overnight?

This workflow sets up a multi-agent content factory inside Discord, where different agents handle research, writing, and visual assets in dedicated channels.

## What It Does

- **Research Agent** scans trending stories, competitor content, and social media for the best content opportunities each morning
- **Writing Agent** takes the top ideas and writes full scripts, threads, or newsletter drafts
- **Thumbnail Agent** generates AI thumbnails or cover images for the content
- Each agent works in its own Discord channel, keeping everything organized and reviewable
- Runs automatically on a schedule (e.g., daily at 8 AM) so you wake up to finished content

## Pain Point

Content creation has three phases — research, writing, and design — and most creators are doing all three manually. Even with AI writing tools, you still have to prompt them one at a time. This system chains agents together in a pipeline where one agent's output feeds the next, completely hands-free.

## Skills You Need

-e 
---

### Custom Morning Brief
File: custom-morning-brief
# Custom Morning Brief

You wake up and spend the first 30 minutes of your day catching up — scrolling news, checking your calendar, reviewing your to-do list, trying to figure out what matters today. What if all of that was already done and waiting for you as a text message?

This workflow has OpenClaw send you a fully customized morning briefing every day at a scheduled time, covering news, tasks, ideas, and proactive recommendations.

## What It Does

- Sends a structured morning report to Telegram, Discord, or iMessage at the same time every day (e.g., 8:00 AM)
- Researches overnight news relevant to your interests by browsing the web
- Reviews your to-do list and surfaces tasks for the day
- Generates creative output (full scripts, email drafts, business proposals — not just ideas) while you sleep
- Recommends tasks the AI can complete autonomously to help you that day

## Pain Point

You're spending your most productive morning hours just getting oriented. Meanwhile, your AI agent sits idle all night. The morning brief turns idle overnight hours into productive prep time — you wake up to work already done.

## Skills You Need

-e 
---

### Daily Reddit Digest
File: daily-reddit-digest
# Daily Reddit Digest
Run a daily digest everyday to give you the top performing posts from your favourite subreddits.

What to use it for:

• Browsing subreddits (hot/new/top posts)
• Searching posts by topic
• Pulling comment threads for context
• Building shortlists of posts to manually review/reply to later

> It's read-only. No posting, voting, or commenting.

## Skills you Need
[reddit-readonly](https://clawhub.ai/buksan1950/reddit-readonly) skill. It doesn't need auth. 

## How to Set it Up
After installing the skill, prompt your OpenClaw:
```text
I want you to give me the top performing posts from the following subreddits.
<paste the list here>
-e 
---

### Daily YouTube Digest
File: daily-youtube-digest
# Daily YouTube Digest

Start your day with a personalized summary of new videos from your favorite YouTube channels — no more missing content from creators you actually want to follow.

## Pain Point

YouTube notifications are unreliable. You subscribe to channels, but their new videos never show up in your home feed. They're not in notifications. They just... disappear. This doesn't mean you don't want to see them — it means YouTube's algorithm buried them.

Plus: it's fun to start the day with curated content insights instead of doom-scrolling a recommendation feed.

## What It Does

- Fetches the latest videos from a list of your favorite channels
- Summarizes or extracts key insights from each video's transcript
- Delivers a digest to you daily (or on demand)

## Skills You Need

Install the [youtube-full](https://clawhub.ai/therohitdas/youtube-full) skill.

-e 
---

### Dynamic Dashboard with Sub-agent Spawning
File: dynamic-dashboard
# Dynamic Dashboard with Sub-agent Spawning

Static dashboards show stale data and require constant manual updates. You want real-time visibility across multiple data sources without building a custom frontend or hitting API rate limits.

This workflow creates a live dashboard that spawns sub-agents to fetch and process data in parallel:

• Monitors multiple data sources simultaneously (APIs, databases, GitHub, social media)
• Spawns sub-agents for each data source to avoid blocking and distribute API load
• Aggregates results into a unified dashboard (text, HTML, or Canvas)
• Updates every N minutes with fresh data
• Sends alerts when metrics cross thresholds
• Maintains historical trends in a database for visualization

## Pain Point

Building a custom dashboard takes weeks. By the time it's done, requirements have changed. Polling multiple APIs sequentially is slow and hits rate limits. You need insight now, not after a weekend of coding.

## What It Does

You define what you want to monitor conversationally: "Track GitHub stars, Twitter mentions, Polymarket volume, and system health." OpenClaw spawns sub-agents to fetch each data source in parallel, aggregates the results, and delivers a formatted dashboard to Discord or as an HTML file. Updates run automatically on a cron schedule.
-e 
---

### AI-Powered Earnings Tracker
File: earnings-tracker
# AI-Powered Earnings Tracker

Following earnings season across dozens of tech companies means checking multiple sources and remembering report dates. You want to stay on top of AI/tech earnings without manually tracking every company.

This workflow automates earnings tracking and delivery:

• Weekly Sunday preview: scans the upcoming earnings calendar and posts relevant tech/AI companies to Telegram
• You pick which companies you care about, and OpenClaw schedules one-shot cron jobs for each earnings date
• After each report drops, OpenClaw searches for results, formats a detailed summary (beat/miss, key metrics, AI highlights), and delivers it

## Skills you Need

- `web_search` (built-in)
- Cron job support in OpenClaw
- Telegram topic for earnings updates

## How to Set it Up

1. Create a Telegram topic called "earnings" for updates.
2. Prompt OpenClaw:
-e 
---

### Event Guest Confirmation
File: event-guest-confirmation
# Event Guest Confirmation

You're hosting an event — a dinner party, a wedding, a company offsite — and you need to confirm attendance from a list of guests. Manually calling 20+ people is tedious: you play phone tag, forget who said what, and lose track of dietary restrictions or plus-ones. Texting works sometimes, but people ignore messages. A real phone call gets a much higher response rate.

This use case has OpenClaw call each guest on your list using the [SuperCall](https://clawhub.ai/xonder/supercall) plugin, confirm whether they're attending, collect any notes, and compile everything into a summary for you.

## What It Does

- Iterates through a guest list (names + phone numbers) and calls each one
- The AI introduces itself as your event coordinator with a friendly persona
- Confirms the event date, time, and location with the guest
- Asks if they're attending, and collects any notes (dietary needs, plus-ones, arrival time, etc.)
- After all calls are complete, compiles a summary: who confirmed, who declined, who didn't pick up, and any notes

## Why SuperCall

This use case works with the [SuperCall](https://clawhub.ai/xonder/supercall) plugin specifically — not the built-in `voice_call` plugin. The key difference: SuperCall is a fully standalone voice agent. The AI persona on the call **only has access to the context you provide** (the persona name, the goal, and the opening line). It cannot access your gateway agent, your files, your other tools, or anything else.

This matters for guest confirmation because:

-e 
---

### Family Calendar Aggregation & Household Assistant
File: family-calendar-household-assistant
# Family Calendar Aggregation & Household Assistant

Modern families juggle five or more calendars — work, personal, shared family, kids' school, extracurriculars — across different platforms and formats. Important events slip through the cracks because no single view exists. Meanwhile, household coordination (grocery lists, pantry inventory, appointment scheduling) happens through scattered text messages that get buried.

This use case turns OpenClaw into an always-on household coordinator: aggregating calendars into a morning briefing, monitoring messages for actionable items, and managing household logistics through a shared chat interface.

## Pain Point

- **Calendar fragmentation**: Work calendars have security restrictions preventing sharing. School calendars arrive as PDFs or hand-written websites. Camp schedules live in emails. Manually checking each one every morning is unsustainable — and "copying events across calendars works well until I forget and one slips through the cracks."
- **Household coordination overhead**: "How much milk do we have?" requires physically checking the fridge, then the basement pantry, then texting back. Multiply this across a week's worth of grocery runs.
- **Missed appointments**: Appointment confirmations arrive via text message and sit there unacted upon — no calendar event, no driving time buffer, no reminder.

## What It Does

- **Morning briefing**: Aggregates all family calendars into a single daily summary delivered via your preferred channel
- **Ambient message monitoring**: Watches iMessage/text conversations and automatically creates calendar events when it detects appointments (dentist confirmations, meeting plans, etc.)
- **Driving time buffers**: Adds travel time blocks before and after detected appointments
- **Household inventory**: Maintains a running inventory of pantry/fridge items that either partner can query from anywhere
- **Grocery coordination**: Deduplicates ingredients across recipes, tracks what's running low, and generates shopping lists
- **Photo-based input**: Snap a photo of a school calendar or freezer contents and the agent processes it into structured data
-e 
---

### Habit Tracker & Accountability Coach
File: habit-tracker-accountability-coach
# Habit Tracker & Accountability Coach

You've tried every habit tracker app out there. They all work for a week, then you stop opening them. The problem isn't the app — it's that tracking habits is passive. What if your agent actively reached out to you, asked how your day went, and adapted its approach based on whether you're on a streak or falling off?

This use case turns OpenClaw into a proactive accountability partner that checks in with you daily via Telegram or SMS.

## Pain Point

Habit apps rely on you remembering to open them. Push notifications are easy to ignore. What actually works for behavior change is **active accountability** — someone (or something) that asks you directly, celebrates your wins, and nudges you when you slip. This agent does exactly that, without the awkwardness of bugging a friend.

## What It Does

- **Daily check-ins** via Telegram or SMS at times you choose (e.g., 7 AM for morning routine, 9 PM for end-of-day review)
- **Tracks habits** you define — exercise, reading, meditation, water intake, coding, whatever matters to you
- **Streak tracking** — knows your current streak for each habit and references it in messages
- **Adaptive nudges** — adjusts tone based on your performance (encouraging when you're consistent, gently persistent when you miss days)
- **Weekly reports** — summarizes your week with completion rates, longest streaks, and patterns (e.g., "You tend to skip workouts on Wednesdays")

## Skills You Need

-e 
---

### Health & Symptom Tracker
File: health-symptom-tracker
# Health & Symptom Tracker

Identifying food sensitivities requires consistent logging over time, which is tedious to maintain. You need reminders to log and analysis to spot patterns.

This workflow tracks food and symptoms automatically:

• Message your food and symptoms in a dedicated Telegram topic and OpenClaw logs everything with timestamps
• 3x daily reminders (morning, midday, evening) prompt you to log meals
• Over time, analyzes patterns to identify potential triggers

## Skills you Need

- Cron jobs for reminders
- Telegram topic for logging
- File storage (markdown log file)

## How to Set it Up

1. Create a Telegram topic called "health-tracker" (or similar).
2. Create a log file: `~/clawd/memory/health-log.md`
-e 
---

### Inbox De-clutter
File: inbox-declutter
# Inbox De-clutter

Newsletters can take up the inbox like nothing else. Often times they pile-up without being opened at all. 

## Skills you Need
[Gmail OAuth Setup](https://clawhub.ai/kai-jar/gmail-oauth).

## How to Set it Up
1. [optional] Create a new gmail specifically for OpenClaw.
2. [optional] Unsubscribe from all newsletters from your main email and subscribe to them using the OpenClaw email.
3. Install the skill and make sure it works. 
4. Instruct OpenClaw:
```txt
I want you to run a cron job everyday at 8 p.m. to read all the newsletter emails of the past 24 hours and give me a digest of the most important bits along with links to read more. Then ask for my feedback on whether you picked good bits, and update your memory based on my preferences for better digests in the future jobs.
```-e 
---

### Personal Knowledge Base (RAG)
File: knowledge-base-rag
# Personal Knowledge Base (RAG)

You read articles, tweets, and watch videos all day but can never find that one thing you saw last week. Bookmarks pile up and become useless.

This workflow builds a searchable knowledge base from everything you save:

• Drop any URL into Telegram or Slack and it auto-ingests the content (articles, tweets, YouTube transcripts, PDFs)
• Semantic search over everything you've saved: "What did I save about agent memory?" returns ranked results with sources
• Feeds into other workflows — e.g., the video idea pipeline queries the KB for relevant saved content when building research cards

## Skills you Need

- [knowledge-base](https://clawhub.ai) skill (or build custom RAG with embeddings)
- `web_fetch` (built-in)
- Telegram topic or Slack channel for ingestion

## How to Set it Up

1. Install the knowledge-base skill from ClawdHub.
2. Create a Telegram topic called "knowledge-base" (or use a Slack channel).
-e 
---

### LaTeX Paper Writing
File: latex-paper-writing
# LaTeX Paper Writing

Setting up a local LaTeX environment is painful — installing TeX Live takes gigabytes, debugging compilation errors is tedious, and switching between your editor and PDF viewer breaks flow. You want to write and compile LaTeX papers conversationally without any local setup.

This workflow turns your agent into a LaTeX writing assistant with instant compilation:

- Write LaTeX collaboratively with the agent — describe what you want and it generates the source
- Compile to PDF instantly with pdflatex, xelatex, or lualatex (no local TeX installation needed)
- Preview PDFs inline without switching to another app
- Use starter templates (article, IEEE, beamer, Chinese article) to skip boilerplate
- Bibliography support with BibTeX/BibLaTeX — just paste your .bib content

## Skills you Need

- [latex-compiler](https://github.com/Prismer-AI/Prismer/tree/main/skills/latex-compiler) skill (4 tools: `latex_compile`, `latex_preview`, `latex_templates`, `latex_get_template`)
- Prismer workspace container (runs the LaTeX server on port 8080 with full TeX Live)

## How to Set it Up

1. Clone and deploy [Prismer](https://github.com/Prismer-AI/Prismer) with Docker (the LaTeX server with full TeX Live starts automatically):
-e 
---

### Local CRM Framework with DenchClaw
File: local-crm-framework
# Local CRM Framework with DenchClaw

Setting up a CRM that actually works with OpenClaw is painful. You need to wire up databases, build UIs, configure browser automation, connect messaging platforms, and somehow get the agent to understand your data schema. Most people give up halfway through and end up with a half-working Notion integration.

DenchClaw is an open-source framework that turns OpenClaw into a fully local CRM, sales automation, and productivity platform — installed with a single command and running entirely on your machine.

## Pain Point

OpenClaw is incredibly powerful as a primitive, but using it for real business workflows (lead tracking, outbound, pipeline management) requires stitching together a dozen tools: a database, a UI, browser access, messaging integrations, file management. Every new integration means more manual setup, more credentials to manage, and more things that break. You want Cursor-level UX for your business operations, not a pile of shell scripts.

## What It Does

- **One-command setup**: `npx denchclaw` installs everything — DuckDB database, web UI, OpenClaw profile, browser automation, skills — and opens at `localhost:3100`
- **Natural language CRM**: Ask "show me companies with more than 5 employees" and it updates the view live. No manual filters needed
- **Full browser automation**: Copies your Chrome profile so the agent has your same auth state — say "import everything from my HubSpot" and it logs in, exports, and imports
- **Multiple views**: Table, Kanban, Calendar, Timeline (Gantt), Gallery, and List views — all configurable by the agent via YAML
- **App builder**: OpenClaw builds self-contained web apps (dashboards, tools, games) that run inside the workspace with access to your data
- **File-system-first**: Table filters, views, column toggles, calendar settings — everything is a file, so OpenClaw can directly read and modify it
- **Works as a coding agent**: DenchClaw built DenchClaw. It's also a full file tree browser and code editor for your Mac

-e 
---

### Market Research & Product Factory
File: market-research-product-factory
# Market Research & Product Factory

You want to build a product but don't know what to build. Or you have a business and need to understand what your customers are struggling with. This workflow uses the Last 30 Days skill to mine Reddit and X for real pain points, then has OpenClaw build solutions to those problems.

## What It Does

- Researches any topic across Reddit and X over the last 30 days using the [Last 30 Days](https://github.com/mvanhorn/last30days-skill/) skill
- Surfaces real challenges, complaints, and feature requests people are posting about
- Helps you identify product opportunities from genuine user pain points
- Takes it a step further: ask OpenClaw to build an MVP that solves one of those challenges
- Creates a full research-to-product pipeline with zero coding on your part

## Pain Point

Most aspiring entrepreneurs struggle with the "what to build" problem. Market research traditionally means hours of manual browsing through forums, social media, and review sites. This automates the entire discovery-to-prototype pipeline.

## Skills You Need

- [Last 30 Days](https://github.com/mvanhorn/last30days-skill/) skill by Matt Van Horde
- Telegram or Discord integration for receiving research reports
-e 
---

### Automated Meeting Notes & Action Items
File: meeting-notes-action-items
# Automated Meeting Notes & Action Items

You just finished a 45-minute team call. Now you need to write up the summary, pull out action items, and distribute them to Jira, Linear, or Todoist — manually. By the time you're done, the next meeting is starting. What if your agent handled all of that the moment the transcript lands?

This use case turns any meeting transcript into structured notes and automatically creates tasks in your project management tool.

## Pain Point

Meeting notes are tedious but critical. Most people either skip them (and lose context) or spend 20+ minutes writing them up. Action items get forgotten because they live in someone's head or buried in a chat thread. This agent eliminates the gap between "we discussed it" and "it's tracked and assigned."

## What It Does

- **Watches** for new meeting transcripts (via Otter.ai export, Google Meet transcript, Zoom recording summary, or a simple paste into chat)
- **Extracts** key decisions, discussion topics, and action items with owners and deadlines
- **Creates tasks** in Jira, Linear, Todoist, or Notion — assigned to the right person with context from the meeting
- **Posts a summary** to Slack or Discord so the whole team has a record
- **Follows up** — optionally pings assignees before deadlines via scheduled reminders

## Skills You Need

-e 
---

### Multi-Agent Specialized Team (Solo Founder Setup)
File: multi-agent-team
# Multi-Agent Specialized Team (Solo Founder Setup)

Solo founders wear every hat — strategy, development, marketing, sales, operations. Context-switching between these roles destroys deep work. Hiring is expensive and slow. What if you could spin up a small, specialized team of AI agents, each with a distinct role and personality, all controllable from a single chat interface?

This use case sets up multiple OpenClaw agents as a coordinated team, each specialized in a domain, communicating through shared memory and controlled via Telegram.

## Pain Point

- **One agent can't do everything well**: A single agent's context window fills up fast when juggling strategy, code, marketing research, and business analysis
- **No specialization**: Generic prompts produce generic outputs — a coding agent shouldn't also be crafting marketing copy
- **Solo founder burnout**: You need a team, not another tool to manage. The agents should work in the background and surface results, not require constant babysitting
- **Knowledge silos**: Insights from marketing research don't automatically inform dev priorities unless you manually bridge them

## What It Does

- **Specialized agents**: Each agent has a distinct role, personality, and model optimized for its domain
- **Shared memory**: Project docs, goals, and key decisions are accessible to all agents — nothing gets lost
- **Private context**: Each agent also maintains its own conversation history and domain-specific notes
- **Single control plane**: All agents are accessible through one Telegram group chat — tag the agent you need
- **Scheduled daily tasks**: Agents proactively work without being asked — content prompts, competitor monitoring, metric tracking
-e 
---

### Multi-Channel Personal Assistant
File: multi-channel-assistant
# Multi-Channel Personal Assistant

Context-switching between apps to manage tasks, schedule events, send messages, and track work is exhausting. You want one interface that routes to all your tools.

This workflow consolidates everything into a single AI assistant:

• Telegram as primary interface with topic-based routing (different topics for video ideas, CRM, earnings, config, etc.)
• Slack integration for team collaboration (task assignment, knowledge base saves, video idea triggers)
• Google Workspace: create calendar events, manage email, upload to Drive — all from chat
• Todoist for quick task capture
• Asana for project management
• Automated reminders: trash day, weekly company letter, etc.

## Skills you Need

- `gog` CLI (Google Workspace)
- Slack integration (bot + user tokens)
- Todoist API or skill
- Asana API or skill
- Telegram channel with multiple topics configured
-e 
---

### Multi-Channel AI Customer Service Platform
File: multi-channel-customer-service
# Multi-Channel AI Customer Service Platform

Small businesses juggle WhatsApp, Instagram DMs, emails, and Google Reviews across multiple apps. Customers expect instant responses 24/7, but hiring staff for round-the-clock coverage is expensive.

This use case consolidates all customer touchpoints into a single AI-powered inbox that responds intelligently on your behalf.

## What It Does

- **Unified inbox**: WhatsApp Business, Instagram DMs, Gmail, and Google Reviews in one place
- **AI auto-responses**: Handles FAQs, appointment requests, and common inquiries automatically
- **Human handoff**: Escalates complex issues or flags them for review
- **Test mode**: Demo the system to clients without affecting real customers
- **Business context**: Trained on your services, pricing, and policies

## Real Business Example

At Futurist Systems, we deploy this for local service businesses (restaurants, clinics, salons). One restaurant reduced response time from 4+ hours to under 2 minutes, handling 80% of inquiries automatically.

## Skills You Need

-e 
---

### Multi-Source Tech News Digest
File: multi-source-tech-news-digest
# Multi-Source Tech News Digest

Automatically aggregate, score, and deliver tech news from 109+ sources across RSS, Twitter/X, GitHub releases, and web search — all managed through natural language.

## Pain Point

Staying updated across AI, open-source, and frontier tech requires checking dozens of RSS feeds, Twitter accounts, GitHub repos, and news sites daily. Manual curation is time-consuming, and most existing tools either lack quality filtering or require complex configuration.

## What It Does

A four-layer data pipeline that runs on a schedule:

1. **RSS Feeds** (46 sources) — OpenAI, Hacker News, MIT Tech Review, etc.
2. **Twitter/X KOLs** (44 accounts) — @karpathy, @sama, @VitalikButerin, etc.
3. **GitHub Releases** (19 repos) — vLLM, LangChain, Ollama, Dify, etc.
4. **Web Search** (4 topic searches) — via Brave Search API

All articles are merged, deduplicated by title similarity, and quality-scored (priority source +3, multi-source +5, recency +2, engagement +1). The final digest is delivered to Discord, email, or Telegram.

The framework is fully customizable — add your own RSS feeds, Twitter handles, GitHub repos, or search queries in 30 seconds.
-e 
---

### OpenClaw + n8n Workflow Orchestration
File: n8n-workflow-orchestration
# OpenClaw + n8n Workflow Orchestration

Letting your AI agent directly manage API keys and call external services is a recipe for security incidents. Every new integration means another credential in `.env.local`, another surface for the agent to accidentally leak or misuse.

This use case describes a pattern where OpenClaw delegates all external API interactions to n8n workflows via webhooks — the agent never touches credentials, and every integration is visually inspectable and lockable.

## Pain Point

When OpenClaw handles everything directly, you get three compounding problems:

- **No visibility**: It's hard to inspect what the agent actually built when it's buried in JavaScript skill files or shell scripts
- **Credential sprawl**: Every API key lives in the agent's environment, one bad commit away from exposure
- **Wasted tokens**: Deterministic sub-tasks (send an email, update a spreadsheet) burn LLM reasoning tokens when they could run as simple workflows

## What It Does

- **Proxy pattern**: OpenClaw writes n8n workflows with incoming webhooks, then calls those webhooks for all future API interactions
- **Credential isolation**: API keys live in n8n's credential store — the agent only knows the webhook URL
- **Visual debugging**: Every workflow is inspectable in n8n's drag-and-drop UI
- **Lockable workflows**: Once a workflow is built and tested, you lock it so the agent can't modify how it interacts with the API
-e 
---

### Goal-Driven Autonomous Tasks
File: overnight-mini-app-builder
# Goal-Driven Autonomous Tasks

Your AI agent is powerful but reactive — it only works when you tell it what to do. What if it knew your goals and proactively came up with tasks to move you closer to them every single day, without being asked?

This workflow turns OpenClaw into a self-directed employee. You brain dump your goals once, and the agent autonomously generates, schedules, and completes tasks that advance those goals — including building you surprise mini-apps overnight.

## What It Does

- You brain dump all your goals, missions, and objectives into OpenClaw (personal and professional)
- Every morning, the agent generates 4-5 tasks it can complete autonomously on your computer
- Tasks go beyond app building: research, writing scripts, building features, creating content, analyzing competitors
- The agent executes the tasks itself and tracks them on a custom Kanban board it builds for you
- You can also have it build you a surprise mini-app every night — a new SaaS idea, a tool that automates a boring part of your life, shipped as an MVP

## Pain Point

Most people have big goals but struggle to break them into daily actionable steps. And even when they do, execution takes all their time. This system offloads both the planning AND the execution to your AI agent. You define the destination; the agent figures out the daily steps and walks them.

## Skills You Need

-e 
---

### Personal CRM with Automatic Contact Discovery
File: personal-crm
# Personal CRM with Automatic Contact Discovery

Keeping track of who you've met, when, and what you discussed is impossible to do manually. Important follow-ups slip through the cracks, and you forget context before important meetings.

This workflow builds and maintains a personal CRM automatically:

• Daily cron job scans email and calendar for new contacts and interactions
• Stores contacts in a structured database with relationship context
• Natural language queries: "What do I know about [person]?", "Who needs follow-up?", "When did I last talk to [person]?"
• Daily meeting prep briefing: before each day's meetings, researches external attendees via CRM + email history and delivers a briefing

## Skills you Need

- `gog` CLI (for Gmail and Google Calendar)
- Custom CRM database (SQLite or similar) or use the [crm-query](https://clawhub.ai) skill if available
- Telegram topic for CRM queries

## How to Set it Up

1. Create a CRM database:
-e 
---

### Phone-Based Personal Assistant
## Pain Point
File: phone-based-personal-assistant
# Phone-Based Personal Assistant

## Pain Point

You want to access your AI agent from any phone without needing a smartphone app or internet browser. You need hands-free voice assistance while driving, walking, or when your hands are occupied.

## What It Does

ClawdTalk enables OpenClaw to receive and make phone calls, turning any phone into a gateway to your AI assistant. You can:
- Call a phone number to speak with your AI agent via voice
- Get calendar reminders, Jira updates, and web search results via voice
- Integrate with Telnyx for reliable phone connectivity

SMS support is coming soon.

## Prompts

```
You are available via phone. When I call, greet me and ask how I can help.

-e 
---

### Phone Call Notifications
File: phone-call-notifications
# Phone Call Notifications

Your agent already monitors things for you — stocks, emails, smart home, calendars — but notifications are easy to ignore. Push notifications pile up. Chat messages get buried. For the stuff that actually matters, you need something you can't swipe away.

This use case gives your agent a phone call as a notification channel. When something is urgent enough, the agent dials your real phone number, tells you what's going on, and you can talk back. Two-way conversation, not a robocall.

## What It Does

- Agent decides something is worth your attention (price alert, urgent email, appointment reminder, anything)
- Agent calls your phone via [clawr.ing](https://clawr.ing), a managed calling service — no Twilio setup, no API keys to configure
- You pick up, hear the alert, and can ask follow-up questions in real time
- Call ends when the conversation is done

The key idea: the agent calls YOU. You don't call the agent. This works with heartbeat checks, cron jobs, or any trigger — the agent evaluates whether something is phone-call-worthy and acts on it.

## Why This Works

OpenClaw agents already have initiative — heartbeat, cron, event triggers. But their output channels are limited to chat platforms you might not be checking. A phone call is the one notification channel that reliably gets your attention, especially when you're away from your desk.

clawr.ing handles the telephony infrastructure. You paste one setup prompt and the agent gains the ability to call. No Twilio account, no phone number provisioning, no webhook configuration. The service covers 100+ countries with real PSTN calls (not VoIP overlays).
-e 
---

### Podcast Production Pipeline
File: podcast-production-pipeline
# Podcast Production Pipeline

You have a podcast idea, maybe even a backlog of episode topics. But between researching guests, writing outlines, drafting intros, generating show notes, and writing social media posts for promotion — the production overhead kills your momentum. What if you handed off a topic and got back a full production package?

This use case chains agents together to handle the entire podcast production workflow from topic to publish-ready assets.

## Pain Point

Solo podcasters and small teams spend more time on production than on actually recording. Research takes hours, show notes are an afterthought, and social media promotion is the first thing that gets skipped. The creative part — the conversation — is maybe 30% of the total effort. This agent handles the other 70%.

## What It Does

- **Episode Research** — given a topic or guest name, compiles background research, talking points, and suggested questions
- **Outline & Script** — generates a structured episode outline with intro script, segment transitions, and closing remarks
- **Show Notes** — after recording, processes the transcript into timestamped show notes with links to everything mentioned
- **Social Media Kit** — creates promotional posts for X, LinkedIn, and Instagram with episode highlights and pull quotes
- **Episode Description** — writes SEO-optimized episode descriptions for Spotify, Apple Podcasts, and YouTube

## Skills You Need

-e 
---

### Polymarket Autopilot: Automated Paper Trading
File: polymarket-autopilot
# Polymarket Autopilot: Automated Paper Trading

Manually monitoring prediction markets for arbitrage opportunities and executing trades is time-consuming and requires constant attention. You want to test and refine trading strategies without risking real capital.

This workflow automates paper trading on Polymarket with custom strategies:

• Monitors market data via API (prices, volume, spreads)
• Executes paper trades using TAIL (trend-following) and BONDING (contrarian) strategies
• Tracks portfolio performance, P&L, and win rate
• Delivers daily summaries to Discord with trade logs and insights
• Learns from patterns: adjusts strategy parameters based on backtesting results

## Pain Point

Prediction markets move fast. Manual trading means missing opportunities, emotional decisions, and difficulty tracking what works. Testing strategies with real money risks losses before you understand market behavior.

## What It Does

The autopilot continuously scans Polymarket for opportunities, simulates trades using configurable strategies, and logs everything for analysis. You wake up to a summary of what it "traded" overnight, what worked, and what didn't.

-e 
---

### Pre-Build Idea Validator
File: pre-build-idea-validator
# Pre-Build Idea Validator

Before OpenClaw starts building anything new, it automatically checks whether the idea already exists across GitHub, Hacker News, npm, PyPI, and Product Hunt — and adjusts its approach based on what it finds.

## What It Does

- Scans 5 real data sources (GitHub, Hacker News, npm, PyPI, Product Hunt) before any code is written
- Returns a `reality_signal` score (0-100) indicating how crowded the space is
- Shows top competitors with star counts and descriptions
- Suggests pivot directions when the space is saturated
- Works as a pre-build gate: high signal = stop and discuss, low signal = proceed

## Pain Point

You tell your agent "build me an AI code review tool" and it happily spends 6 hours coding. Meanwhile, 143,000+ repos already exist on GitHub — the top one has 53,000 stars. The agent never checks because you never asked, and it doesn't know to look. You only discover competitors after you've invested significant time. This pattern repeats for every new project idea.

## Skills You Need

- [idea-reality-mcp](https://github.com/mnemox-ai/idea-reality-mcp) — MCP server that scans real data sources and returns a competition score

-e 
---

### Project State Management System: Event-Driven Alternative to Kanban
File: project-state-management
# Project State Management System: Event-Driven Alternative to Kanban

Traditional Kanban boards are static and require manual updates. You forget to move cards, lose context between sessions, and can't track the "why" behind state changes. Projects drift without clear visibility.

This workflow replaces Kanban with an event-driven system that tracks project state automatically:

• Stores project state in a database with full history
• Captures context: decisions, blockers, next steps, key insights
• Event-driven updates: "Just finished X, blocked on Y" → automatic state transition
• Natural language queries: "What's the status of [project]?", "Why did we pivot on [feature]?"
• Daily standup summaries: What happened yesterday, what's planned today, what's blocked
• Git integration: links commits to project events for traceability

## Pain Point

Kanban boards become stale. You waste time updating cards instead of doing work. Context gets lost—three months later, you can't remember why you made a key decision. There's no automatic link between code changes and project progress.

## What It Does

Instead of dragging cards, you chat with your assistant: "Finished the auth flow, starting on the dashboard." The system logs the event, updates project state, and preserves context. When you ask "Where are we on the dashboard?" it gives you the full story: what's done, what's next, what's blocking you, and why.
-e 
---

### Second Brain
File: second-brain
# Second Brain

You come up with ideas, find interesting links, hear about books to read — but you never have a good system for capturing them. Notion gets complex, Apple Notes becomes a graveyard of 10,000 unread entries. You need something as simple as texting a friend.

This workflow turns OpenClaw into a memory-capture system you interact with via text message, backed by a custom searchable UI you can browse anytime.

## What It Does

- Text anything to your OpenClaw via Telegram, iMessage, or Discord — "Remind me to read a book about local LLMs" — and it remembers it instantly
- OpenClaw's built-in memory system stores everything you tell it permanently
- A custom Next.js dashboard lets you search through every memory, conversation, and note
- Global search (Cmd+K) across all memories, documents, and tasks
- No folders, no tags, no complex organization — just text and search

## Pain Point

Every note-taking app eventually becomes a chore. You stop using it because the friction of organizing is higher than the friction of forgetting. The key insight is: **capture should be as easy as texting, and retrieval should be as easy as searching**.

## Skills You Need

-e 
---

### Self-Healing Home Server & Infrastructure Management
File: self-healing-home-server
# Self-Healing Home Server & Infrastructure Management

Running a home server means being on-call 24/7 for your own infrastructure. Services go down at 3 AM, certificates expire silently, disk fills up, and pods crash-loop — all while you're asleep or away.

This use case turns OpenClaw into a persistent infrastructure agent with SSH access, automated cron jobs, and the ability to detect, diagnose, and fix issues before you know there's a problem.

## Pain Point

Home lab operators and self-hosters face a constant maintenance burden:

- Health checks, log monitoring, and alerting require manual setup and attention
- When something breaks, you have to SSH in, diagnose, and fix — often from your phone
- Infrastructure-as-code (Terraform, Ansible, Kubernetes manifests) needs regular updates
- Knowledge about your setup lives in your head, not in searchable documentation
- Routine tasks (email triage, deployment checks, security audits) eat hours every week

## What It Does

- **Automated health monitoring**: Cron-based checks on services, deployments, and system resources
- **Self-healing**: Detects issues via health checks and applies fixes autonomously (restart pods, scale resources, fix configs)
-e 
---

### Semantic Memory Search
File: semantic-memory-search
# Semantic Memory Search

OpenClaw's built-in memory system stores everything as markdown files — but as memories grow over weeks and months, finding that one decision from last Tuesday becomes impossible. There is no search, just scrolling through files.

This use case adds **vector-powered semantic search** on top of OpenClaw's existing markdown memory files using [memsearch](https://github.com/zilliztech/memsearch), so you can instantly find any past memory by meaning, not just keywords.

## What It Does

- Index all your OpenClaw markdown memory files into a vector database (Milvus) with a single command
- Search by meaning: "what caching solution did we pick?" finds the relevant memory even if the word "caching" does not appear
- Hybrid search (dense vectors + BM25 full-text) with RRF reranking for best results
- SHA-256 content hashing means unchanged files are never re-embedded — zero wasted API calls
- File watcher auto-reindexes when memory files change, so the index is always up to date
- Works with any embedding provider: OpenAI, Google, Voyage, Ollama, or fully local (no API key needed)

## Pain Point

OpenClaw's memory is stored as plain markdown files. This is great for portability and human readability, but it has no search. As your memory grows, you either have to grep through files (keyword-only, misses semantic matches) or load entire files into context (wastes tokens on irrelevant content). You need a way to ask "what did I decide about X?" and get the exact relevant chunk, regardless of phrasing.

## Skills You Need
-e 
---

### Todoist Task Manager: Agent Task Visibility
File: todoist-task-manager
# Todoist Task Manager: Agent Task Visibility
Maximize transparency for long-running agentic workflows by syncing internal reasoning and progress logs directly to Todoist.

## Pain Point
When agents run complex, multi-step tasks (like building a full-stack app or performing deep research), the user often loses track of what the agent is currently doing, what steps have been completed, and where the agent might be stuck. Checking chat logs manually is tedious for background tasks.

## What It Does
This use case uses the `todoist-task-manager` skill to:
1.  **Visualize State**: Create tasks in specific sections like `🟡 In Progress` or `🟠 Waiting`.
2.  **Externalize Reasoning**: Post the agent's internal "Plan" into the task description.
3.  **Stream Logs**: Add sub-step completions as comments to the task in real-time.
4.  **Auto-Reconcile**: A heartbeat script checks for stalled tasks and notifies the user.

## Skills you Need
You don't need a pre-built skill. Simply prompt your OpenClaw agent to create the bash scripts described in the **Setup Guide** below. Since OpenClaw can manage its own filesystem and execute shell commands, it will effectively "build" the skill for you upon request.

## Detailed Setup Guide

### 1. Configure Todoist
Create a project (e.g., "OpenClaw Workspace") and get its ID. Create sections for different states:
-e 
---

### X Account Analysis
File: x-account-analysis
# X Account Analysis

There are many websites designed to give you a qualitative analysis of your X account. While X already gives you an **analytics** section, it's more focused to show your numbers on your performance.

But a qualitative analysis focuses on the quality of your posts, not the performance stats. Some insights you can get from this type of analysis:
- What are the patterns that make my posts go viral?
- What topics I talk about get me most engagement?
- Why do I get posts with 1000+ likes but sometimes posts with <5 likes? What am I doing wrong?

There are many websites and apps designed to give you X analytics, but they focus on the statistics. There are probably 1-2 websites that let you talk with an AI to understand your performance. 

But now you can use OpenClaw to do this analysis for you, without needing to pay $10-$50 for subscriptions on these websites.

## Skills you Need
Bird Skill. `clawhub install bird` (it comes pre-bundled)

## How to Set it Up
Here's the flow:
1. Make sure Bird skill is working.
2. For security and isolation, you better create a new account for your ClawdBot.
-e 
---

### X/Twitter Automation from Chat
File: x-twitter-automation
# X/Twitter Automation from Chat

Full X/Twitter automation through natural language — post tweets, reply, like, retweet, follow, DM, search, extract data, run giveaways, and monitor accounts, all from your OpenClaw chat.

## Pain Point

Managing an X/Twitter presence requires jumping between the app, third-party dashboards, and analytics tools. Running giveaways means manual winner picking. Extracting followers, likers, or retweeters requires scraping scripts. There is no single interface that lets you do all of this conversationally.

## What It Does

TweetClaw is an OpenClaw plugin that connects your agent to the X/Twitter API. You interact entirely through chat:

- **Post & engage** — Compose tweets, reply to threads, like, retweet, follow/unfollow, send DMs
- **Search & extract** — Search tweets and users, extract followers, likers, retweeters, quote tweeters, list members
- **Giveaways** — Pick random winners from tweet engagements with configurable filters (minimum followers, account age, keyword requirements)
- **Monitors** — Watch accounts for new tweets or follower changes and get notified

All actions go through a managed API — no browser cookies, no scraping, no credential exposure.

## Prompts
-e 
---

### YouTube Content Pipeline
File: youtube-content-pipeline
# YouTube Content Pipeline

As a daily YouTube creator, finding fresh, timely video ideas across the web and X/Twitter is time-consuming. Tracking what you've already covered prevents duplicates and helps you stay ahead of trends.

This workflow automates the entire content scouting and research pipeline:

• Hourly cron job scans breaking AI news (web + X/Twitter) and pitches video ideas to Telegram
• Maintains a 90-day video catalog with view counts and topic analysis to avoid re-covering topics
• Stores all pitches in a SQLite database with vector embeddings for semantic dedup (so you never get pitched the same idea twice)
• When you share a link in Slack, OpenClaw researches the topic, searches X for related posts, queries your knowledge base, and creates an Asana card with a full outline

## Skills you Need

- `web_search` (built-in)
- [x-research-v2](https://clawhub.ai) or custom X/Twitter search skill
- [knowledge-base](https://clawhub.ai) skill for RAG
- Asana integration (or Todoist)
- `gog` CLI for YouTube Analytics
- Telegram topic for receiving pitches

-e 
---


---

# Additional Use Cases — From YouTube & Community Sources

Source: YouTube (miJLo234L9s), hostinger.com, quantumbyte.ai, ucstrategies.com

---

## Personal Productivity & Daily Automation

### 16. Garmin Health Integration + Smart Reminders
**Description:** Syncs Garmin watch data — sleep, steps, heart rate — into daily brief. AI analyzes patterns and sends personalized health nudges.
**Prompt example:** "Check my Garmin sleep score. If below 70, suggest tonight's bedtime. If above 85, suggest an intensity workout."
**Tools:** Garmin API, cron job, Telegram notification
**Dream AI angle:** Personal trainer AI upsell

### 17. 365-Day Meal Planning & Shopping Lists
**Description:** Builds meal plans based on preferences, weather, schedule. Auto-generates shopping lists organized by store/aisle. Deduplicates items. Adjusts for weather (grilling nights when sunny).
**Prompt example:** "Build this week's meal plan. We have chicken, pasta, and veggies. Add missing items to the shopping list organized by Costco and Whole Foods aisles."
**Tools:** Web search, calendar integration, weather API
**Dream AI angle:** Lifestyle automation, smart home vertical

### 18. Voice Note → Journal Entry
**Description:** Records voice memos throughout the day. AI transcribes, formats into journal entries, extracts action items, updates daily notes in Obsidian.
**Prompt example:** "Process today's voice notes. Add journal entries to Obsidian, extract tasks to Todoist, and create tomorrow's priority list."
**Tools:** STT, Obsidian API, Todoist API
**Dream AI angle:** Productivity AI Employee

### 19. Weekly Review from Transcripts
**Description:** Pulls meeting transcripts, calendar events, and notes. Generates a weekly review with wins, metrics, and next week's priorities.
**Prompt example:** "It's Friday 5 PM. Generate my weekly review: what I accomplished, what's blocked, revenue moved, and top 3 priorities for next week."
**Tools:** Calendar, meeting transcripts, email
**Dream AI angle:** CEO dashboard / morning brief

### 20. School Test & Event Reminders
**Description:** Monitors school calendars and kids' schedules. Sends smart reminders: "Emma has a math test Thursday — suggest she review chapters 3-4 tonight."
**Prompt example:** "What's coming up this week for the kids? Anything we need to prepare for?"
**Tools:** Shared calendar, notification system
**Dream AI angle:** Family assistant, personal vertical

## Business & Professional (NEW)

### 21. Competitor Pricing Monitor
**Description:** Daily scan of competitor pricing pages, product updates, and public metrics. Sends a competitive intelligence digest.
**Prompt example:** "Scan competitor A, B, C pricing pages. Flag any changes from last week. Highlight if any undercut our $497 tier."
**Tools:** Firecrawl, cron, Telegram digest
**Dream AI angle:** Scout service / competitive intelligence

### 22. Client Onboarding Automation
**Description:** When a deal closes in CRM: auto-create client folder, send welcome email, schedule kickoff call, create project workspace, notify team.
**Trigger:** CRM deal → "won" status
**Steps:** Create Google Drive folder → Generate contract → Send welcome email → Book kickoff → Create Notion project → Notify team in Slack
**Tools:** CRM webhook, Google Drive API, Calendly, Notion, Slack
**Dream AI angle:** Client onboarding AI Employee

### 23. Invoice Matching & Payment Processing
**Description:** Receives invoices via email, extracts details (amount, vendor, PO number), matches against purchase orders, flags discrepancies, processes for payment.
**Prompt example:** "Process today's invoices. Match against POs. Flag anything over budget or missing a PO. Ready for payment approval by 3 PM."
**Tools:** Email monitoring, OCR, accounting software API
**Dream AI angle:** Finance AI Employee

### 24. Website Traffic & Conversion Monitoring
**Description:** Hourly check of website visitors, conversion rates, page performance. Alerts on anomalies (traffic spike/drop, broken pages, slow load times).
**Prompt example:** "Check website status. Any traffic anomalies? How are landing pages performing? Any 404 errors?"
**Tools:** GA4 API, Uptime monitoring, Lighthouse
**Dream AI angle:** Analytics AI + performance monitoring

### 25. Code Deploy Pipeline
**Description:** Deploys code from GitHub releases to production. Runs tests, checks changelog, deploys, verifies health, and notifies team.
**Trigger:** GitHub release published
**Steps:** Run tests → Build → Deploy → Health check → Notify
**Tools:** GitHub Actions, deploy scripts, health checks
**Dream AI angle:** BOLT automation services

### 26. Reddit Problem Monitoring
**Description:** Monitors specific subreddits for posts describing problems your business solves. Drafts helpful (non-spammy) responses for human review.
**Prompt example:** "Scan r/realestate for posts about 'missing leads' or 'after hours calls'. Draft helpful replies — DO NOT post without approval."
**Tools:** Reddit API, AI draft, Telegram approval
**Dream AI angle:** Lead generation + social selling

### 27. Earthquake / Emergency Alert Monitor
**Description:** Monitors earthquake alerts, weather warnings, or business-critical systems. Sends immediate notifications with severity and recommended actions.
**Prompt example:** "Monitor USGS for earthquakes within 100 miles of our warehouse. If M3.0+, alert immediately with impact assessment."
**Tools:** USGS API, cron, notification system
**Dream AI angle:** IoT / smart business monitoring

### 28. Content Calendar Auto-Scheduler
**Description:** Proactively schedules content based on trends, competitor activity, and audience engagement patterns. Researches trending stories, generates ideas, creates drafts.
**Prompt example:** "What's trending in real estate this week? Generate 5 LinkedIn post ideas. Draft the best one. Schedule for Tuesday 8 AM."
**Tools:** Web search, content generation, scheduling
**Dream AI angle:** Content factory / marketing AI Employee

### 29. CRM Pipeline Hygiene
**Description:** Daily scan of CRM pipeline. Flags stale deals, missing follow-ups, deals stuck too long in one stage. Drafts re-engagement emails for human review.
**Prompt example:** "Scan my CRM. Any deals stuck in 'Proposal' stage >7 days? Draft re-engagement emails. Flag any deal at risk of going cold."
**Tools:** CRM API, email draft, daily cron
**Dream AI angle:** Sales AI Employee / FORGE integration

### 30. Obsidian Daily Notes Manager
**Description:** Maintains daily notes in Obsidian. Creates new daily note, links to previous/next, pulls in calendar events, tasks, and meeting notes.
**Prompt example:** "Create today's daily note. Link to yesterday's note. Pull in today's calendar and any tasks due. Add section for standup notes."
**Tools:** Obsidian API (local files), calendar, task manager
**Dream AI angle:** Knowledge management / Second Brain

### 31. Multi-Source Research Agent
**Description:** Spawns multiple sub-agents to research a topic from different angles simultaneously. One researches competitors, one finds news, one checks social sentiment, one analyzes pricing. Synthesizes into one report.
**Prompt example:** "Research the AI voice assistant market. Agent 1: top 5 competitors + pricing. Agent 2: latest news and trends. Agent 3: Reddit/Twitter sentiment. Agent 4: gaps we could fill. Combine into one brief."
**Tools:** Multi-agent spawning, web search, synthesis
**Dream AI angle:** Scout service / market research product

### 32. Automated Guest Confirmation (Events)
**Description:** For events/restaurant/appointment businesses: auto-calls or texts guests 24h before, confirms attendance, updates count, sends reminders.
**Prompt example:** "Tomorrow's dinner event: 50 guests. Text everyone at 10 AM to confirm. Update count. Send parking info to confirmed guests."
**Tools:** SMS/Voice API, calendar, CRM
**Dream AI angle:** Event management AI Employee

### 33. WhatsApp Business AI Assistant
**Description:** WhatsApp Business account with AI that answers customer questions, takes orders, sends catalog info, and escalates complex queries to humans.
**Prompt example:** "Customer asks about pricing for kitchen renovation. Send catalog PDF, ask about timeline, offer free estimate booking."
**Tools:** WhatsApp Business API, knowledge base, calendar
**Dream AI angle:** Multi-channel AI Employee

### 34. Email Triage & Priority Scoring
**Description:** Scans all incoming emails, scores by urgency/importance, creates priority queue, drafts responses for high-priority, summarizes low-priority into digest.
**Prompt example:** "Scan my inbox. Score emails 1-5 by urgency. Draft responses for anything score 4+. Summarize the rest into a daily digest."
**Tools:** Gmail API, AI scoring, Telegram/email delivery
**Dream AI angle:** Email AI Employee

### 35. Smart Appointment Optimizer
**Description:** Analyzes appointment schedule, finds gaps, suggests optimal booking slots, reschedules low-priority meetings to free up high-value time blocks.
**Prompt example:** "Review my calendar this week. Move any internal meetings to low-priority slots. Keep 9-11 AM clear for client calls."
**Tools:** Calendar API, scheduling optimization
**Dream AI angle:** Productivity AI Employee

### 36. Social Media Sentiment Monitor
**Description:** Tracks brand mentions across Twitter, Reddit, Instagram, Facebook. Analyzes sentiment, flags negative threads, surfaces positive testimonials for resharing.
**Prompt example:** "Any brand mentions today? Sentiment analysis across all platforms. Alert me if negative sentiment >30%. Screenshot best testimonial."
**Tools:** Social APIs, sentiment analysis, notification
**Dream AI angle:** Reputation management AI

### 37. Inventory Alert System
**Description:** Monitors stock levels, predicts when items will run out based on sales velocity, auto-reorders or alerts when below threshold.
**Prompt example:** "Check inventory levels. Product A has 12 days of stock at current velocity. Auto-order if below 14-day threshold."
**Tools:** Inventory API, sales data, email/SMS
**Dream AI angle:** Operations AI Employee

### 38. SEO Audit & Auto-Fix
**Description:** Weekly scan of all website pages for SEO issues: broken links, missing meta tags, slow pages, missing alt text. Auto-generates fixes for review.
**Prompt example:** "Run SEO audit on dreamai.com. Any broken links? Missing meta descriptions? Pages over 3-second load time? Fix what you can, flag the rest."
**Tools:** Screaming Frog, Lighthouse, CMS API
**Dream AI angle:** Website management AI

### 39. Automated Testimonial Capture
**Description:** Post-service automated follow-up asking for testimonial. If positive, formats for website/social. If negative, alerts owner for resolution.
**Prompt example:** "3 days after service completion: text customer asking for feedback. If 4-5 stars, ask for permission to use as testimonial. Format for website."
**Tools:** CRM trigger, SMS, rating system
**Dream AI angle:** Review Rocket Kit integration

### 40. Board/Investor Update Generator
**Description:** Gathers KPIs, wins, metrics, and pipeline data. Generates formatted board update or investor report with charts and narrative.
**Prompt example:** "Generate Q1 board update: revenue, customer count, churn rate, top 3 wins, top 3 risks, and asks. Use data from our dashboard."
**Tools:** Analytics APIs, chart generation, document creation
**Dream AI angle:** Executive AI assistant

### 41. Automated Referral Follow-Up
**Description:** Monitors client milestones (30/60/90 days post-close). Sends referral requests at optimal times. Tracks referral pipeline.
**Prompt example:** "Client John Smith hit 90-day mark yesterday. Send referral request: personal, warm, offer $500 credit for any closed referral."
**Tools:** CRM, email/SMS automation
**Dream AI angle:** Client retention / PULSE integration

### 42. Employee Onboarding AI
**Description:** Guides new hires through onboarding: paperwork, training videos, system access, introductions, first-week schedule. Checks in daily.
**Prompt example:** "New hire starts Monday. Send welcome email with first-day details. Schedule all onboarding meetings. Check in daily for first week."
**Tools:** HR system, calendar, email, Slack
**Dream AI angle:** HR AI Employee

### 43. Property Listing Auto-Generator
**Description:** Takes property details (photos, specs, address) and auto-generates: MLS description, social media posts, email announcement, flyer text, virtual tour script.
**Prompt example:** "New listing: 3BR/2BA, 1800 sqft, pool, recently renovated. Generate: MLS description, 3 Instagram posts, email blast to buyer list, open house flyer."
**Tools:** Template engine, AI writing, image processing
**Dream AI angle:** Real Estate AI Playbook feature

### 44. Contract Renewal Tracker
**Description:** Monitors all contracts (client, vendor, subscription) and alerts 30/60/90 days before renewal. Drafts renewal proposals or cancellation notices.
**Prompt example:** "3 contracts renewing in 60 days: Client A ($2K/mo), Vendor B ($500/mo), Software C ($200/mo). Draft renewal proposals for A, evaluate B for replacement."
**Tools:** Contract database, calendar, email
**Dream AI angle:** Operations AI Employee

### 45. Dynamic Pricing Monitor
**Description:** Tracks competitor pricing in real-time. Adjusts recommendations based on market position, demand, and cost floor.
**Prompt example:** "Competitor A dropped price to $397. We're at $497. Options: match (-$100/mo), differentiate (add bonus), or hold (our reviews are stronger)."
**Tools:** Web scraping, price tracking, AI analysis
**Dream AI angle:** Competitive intelligence

---

## Summary: Total Use Case Database

| Source | Count |
|--------|-------|
| awesome-openclaw-usecases | 40 |
| YouTube/community (this file) | 30 |
| **Total unique use cases** | **70+** |

## Dream AI Sales Angle

Every use case = a potential AI Employee deployment.
- Simple use cases → $297 Starter Kit
- Medium complexity → $497/mo Managed Service
- Complex/multi-agent → $1,497+ Custom Deployment
- Vertical-specific → Niche Playbook ($17-27)

The use case database is Dream AI's strongest sales tool: it shows prospects exactly what their business could look like with AI.

---

# Additional Use Cases — useclaw.vercel.app & TLDL Survey (100+ users)

Source: tldl.io/blog/openclaw-use-cases-2026, substack (aiblewmymind), simplified.com, hostinger.com

---

## Content Automation (35% adoption, 4.5/5 satisfaction)

### 46. X/Twitter Auto-Poster
**Description:** Connects blog RSS → generates platform-specific posts → learns your writing style → auto-publishes. Saves 10+ hours/week.
**Prompt:** "When I publish a new blog post, create 3 X posts from it: one hook tweet, one thread, one quote-style. Match my tone: direct, slightly contrarian, tech-savvy."
**Dream AI:** Social Media Strategy bundle

### 47. LinkedIn Thought Leadership Engine
**Description:** Monitors industry news, drafts posts in your voice, schedules for optimal times, engages with comments.
**Prompt:** "Every Monday morning, find the 3 biggest AI industry stories from last week. Write a LinkedIn post about each with my take. Post at 8 AM, 12 PM, and 5 PM."
**Dream AI:** Social Media Strategy bundle

### 48. Newsletter Writing Pipeline
**Description:** Researches topic → drafts content → maintains style consistency → avoids repetition from previous issues → schedules.
**Prompt:** "Write this week's newsletter about [topic]. Reference last 4 issues to avoid overlap. Keep tone conversational. Include 3 data points. Schedule for Tuesday 8 AM."
**Dream AI:** Content Calendar / QUILL

### 49. YouTube Video Summarizer (Daily Cron)
**Description:** Transcribes videos from favorite channels → extracts key insights → saves to note-taking system → sends digest.
**Prompt:** "Summarize the latest videos from [channels]. Extract 3 key takeaways each. Add to my Obsidian vault and send me a digest in Telegram."
**Dream AI:** Content repurposing

### 50. SEO Blog Post Generator
**Description:** Pulls data from multiple sources → structures around target keywords → drafts post → light editing needed.
**Prompt:** "Write a 1,500-word blog post about 'AI for real estate agents' targeting keyword 'ai real estate automation'. Include stats, examples, and a CTA for our Starter Kit."
**Dream AI:** Content factory

### 51. Social Media Style Learner
**Description:** Learns your writing style from past posts → generates new content in your voice → platform-specific adaptations.
**Prompt:** "Analyze my last 50 LinkedIn posts. Learn my style: sentence length, humor level, CTA patterns. Now write 10 new posts in that voice about AI automation."
**Dream AI:** Content factory

### 52. Podcast Show Notes Generator
**Description:** Transcribes podcast → generates show notes, timestamps, key quotes, social media clips, promotional copy.
**Prompt:** "Transcribe this episode. Generate: show notes with timestamps, 5 key quotes, 3 social media posts, and an email summary for subscribers."
**Dream AI:** Content repurposing

---

## Research & Data (28% adoption, 4.3/5 satisfaction)

### 53. AI News Aggregation (500+ sources)
**Description:** Monitors hundreds of RSS feeds, Twitter accounts, and news sites. Delivers curated daily digest tailored to your interests.
**Prompt:** "Monitor these 50 AI/tech news sources. Every morning at 7 AM, send me the top 5 stories with 2-sentence summaries and why they matter for my business."
**Dream AI:** Market research / Scout

### 54. Competitor Weekly Analysis
**Description:** Scrapes competitor websites for product changes, pricing updates, blog posts, and news. Formats into structured weekly report.
**Prompt:** "Every Monday, scan competitor A, B, C websites. Check: new pages, pricing changes, new features, blog posts. Format into a competitive intelligence brief."
**Dream AI:** Scout service

### 55. Reddit Pain Point Mining
**Description:** Monitors specific subreddits for complaints, problems, and unmet needs. Surfaces product opportunities.
**Prompt:** "Scan r/smallbusiness, r/entrepreneur for posts about 'missed calls', 'phone tag', 'scheduling nightmare'. Extract top 10 pain points this week. Draft product angle for each."
**Dream AI:** Lead gen / product development

### 56. SEC Earnings Tracker
**Description:** Monitors SEC filings, earnings calls, and press releases for specific companies. Alerts on relevant changes.
**Prompt:** "Monitor SEC filings for [list of companies]. Alert me immediately if: revenue guidance changes, new product lines announced, or executive changes. Include stock impact context."
**Dream AI:** Internal tool / financial vertical

### 57. Industry Trend Radar
**Description:** Tracks trends across news, social, patents, academic papers. Produces monthly trend report with opportunity assessment.
**Prompt:** "Monthly report: what are the top 10 trends in the AI automation space? Which ones create opportunities for Dream AI? Include market size data where available."
**Dream AI:** Scout

### 58. Patent Monitor
**Description:** Scans patent databases for filings in your industry. Alerts on potentially disruptive technologies.
**Prompt:** "Monitor USPTO for new patents in 'AI automation', 'voice AI', 'customer service automation'. Flag any patents from major tech companies that could affect our market."
**Dream AI:** Scout

### 59. Job Market Analyzer
**Description:** Monitors job postings in your industry to identify hiring trends, skill demands, and potential partnership opportunities.
**Prompt:** "This week's job postings analysis: how many companies are hiring for 'AI automation', 'chatbot', 'virtual assistant' roles? What's the salary range? These companies might be Dream AI prospects."
**Dream AI:** Lead gen / Scout

### 60. Domain / Business Acquisition Research
**Description:** Researches businesses or domains for potential acquisition. Analyzes revenue, traffic, competitors, asking price fairness.
**Prompt:** "Analyze [business/domain]: estimated revenue, traffic trends, growth trajectory, competitor landscape, fair valuation. Is it worth the $X asking price?"
**Dream AI:** Business development

---

## Productivity (20% adoption, 4.0/5 satisfaction)

### 61. Smart Calendar Conflict Resolver
**Description:** Detects calendar conflicts → proposes resolutions → sends meeting updates → blocks focus time based on workload.
**Prompt:** "Check my calendar this week. Any conflicts? Block 2-hour focus blocks on Wednesday and Thursday mornings. Reschedule any internal meetings that conflict."
**Dream AI:** Personal AI Employee

### 62. Document Q&A (Legal/Sales)
**Description:** Chat with your notes, contracts, and documents. Legal teams search contracts, sales teams query pitch decks.
**Prompt:** "Based on our contract with [Client X], what's the renewal date? Any auto-renewal clauses? What are the termination terms?"
**Dream AI:** Knowledge base / Second Brain

### 63. Meeting Action Item Extractor
**Description:** Transcribes meetings → extracts action items → assigns to participants → follows up if not completed.
**Prompt:** "Review today's meeting transcript. Extract all action items. Create Todoist tasks for each with owners and deadlines. Email summary to all participants."
**Dream AI:** Productivity AI Employee

### 64. Travel Itinerary Builder
**Description:** Builds complete travel plans based on preferences, budget, calendar. Books flights, hotels, restaurants.
**Prompt:** "Plan my NYC trip March 15-18. Budget $2,000. I need: 3 client dinners (upscale), 1 conference, downtime for work. Book flights from Austin. Find hotel near Midtown."
**Dream AI:** Personal assistant

### 65. Weekly Time Audit
**Description:** Analyzes calendar, email, and activity logs to show where time actually goes. Identifies optimization opportunities.
**Prompt:** "Analyze last week: how much time did I spend in meetings vs. deep work? What meetings could have been emails? Suggest 3 time-saving changes for this week."
**Dream AI:** Productivity / METRIC

### 66. Smart Follow-Up Scheduler
**Description:** After every meeting/call, automatically schedules follow-up tasks and reminders based on what was discussed.
**Prompt:** "That call with [Client] just ended. Schedule: follow-up email tomorrow, send proposal by Friday, check-in call next Wednesday. Add all to CRM."
**Dream AI:** Sales AI Employee / FORGE

---

## Business Operations (varying adoption, high satisfaction)

### 67. Customer Support Triage (70% autonomous)
**Description:** Monitors support inbox → categorizes tickets → answers FAQs → routes complex issues → updates status. Handles 70% without human.
**Prompt:** "Monitor support@dreamai.com. Auto-answer: pricing questions (send pricing page), technical setup (send docs), billing (escalate to finance). Flag anything else for human review."
**Dream AI:** Customer Success AI Employee

### 68. Invoice OCR + Accounting Entry
**Description:** Reads invoices via OCR → extracts vendor, amount, PO number → matches against purchase orders → enters into accounting system.
**Prompt:** "Process today's invoices from email. Match against POs. Flag anything over budget or without a PO. Ready for approval by 3 PM."
**Dream AI:** Finance AI Employee

### 69. Sales Call → CRM Auto-Log
**Description:** Transcribes sales calls → extracts key info (needs, objections, next steps) → logs to Salesforce/HubSpot automatically. Saves 15-20 min per call.
**Prompt:** "That sales call with [Prospect] just ended. Log to CRM: key needs, objections raised, proposed solution, next steps, follow-up date. Update deal stage."
**Dream AI:** Sales AI Employee / FORGE

### 70. Proposal Auto-Generator
**Description:** Takes client requirements + past proposals → generates custom proposal with pricing, timeline, scope → sends for approval.
**Prompt:** "Generate proposal for [Client]: they need AI receptionist + CRM integration. Use our Pro tier ($497/mo). Include: scope, timeline, deliverables, terms. Send to me for review before sending to client."
**Dream AI:** AXIOM / sales enablement

### 71. Contract Expiry Monitor
**Description:** Tracks all contracts (clients, vendors, software) and alerts 30/60/90 days before renewal. Drafts renewal or cancellation notices.
**Prompt:** "Review all active contracts. Any expiring in 60 days? Draft renewal proposals for clients, evaluate vendor contracts for renegotiation or cancellation."
**Dream AI:** Operations AI Employee

### 72. Employee Onboarding Checklist
**Description:** Guides new hires through complete onboarding: paperwork, system access, training schedule, introductions, first-week tasks.
**Prompt:** "[New hire] starts Monday. Send welcome email, create accounts, schedule orientation meetings, assign buddy, generate first-week task list."
**Dream AI:** HR AI Employee

### 73. Quarterly Business Review Generator
**Description:** Pulls KPIs, revenue, customer data, and team performance. Generates formatted QBR document with charts and narrative.
**Prompt:** "Generate Q1 business review: revenue vs. target, customer acquisition cost, churn rate, NPS, top wins, top risks, and Q2 priorities. Pull data from our dashboard."
**Dream AI:** METRIC / executive tool

---

## Software Development (15% adoption, 4.8/5 satisfaction)

### 74. Automated PR Review
**Description:** Reviews every pull request for style violations, potential bugs, security issues before human review.
**Prompt:** "Review this PR: check for common issues, style violations, potential bugs, security vulnerabilities. Summarize findings and recommend approve/request changes."
**Dream AI:** BOLT / development services

### 75. Bug Triage & Auto-Assignment
**Description:** Categorizes incoming bugs by severity and affected components. Auto-assigns to appropriate team members.
**Prompt:** "Review new bug reports. Categorize by: severity (P0-P3), component, affected users. Assign P0 to on-call, P1-P2 to component owner."
**Dream AI:** BOLT

### 76. Auto Documentation from Code
**Description:** Reads code comments and function signatures → generates API documentation → keeps docs in sync with code changes.
**Prompt:** "Generate API documentation from our codebase. Focus on the new endpoints added this sprint. Format as OpenAPI spec."
**Dream AI:** BOLT / development services

### 77. Automated Test Generation
**Description:** Writes test cases based on code changes. Handles repetitive tests so humans focus on edge cases.
**Prompt:** "Review this week's code changes. Generate unit tests for new functions. Aim for 80% coverage on modified code."
**Dream AI:** BOLT

### 78. Codebase Migration Planner
**Description:** Analyzes existing codebase → identifies migration paths → creates step-by-step migration plan → estimates effort.
**Prompt:** "Analyze our codebase. Create migration plan from Express to Fastify. Include: effort estimate, breaking changes, testing strategy, rollback plan."
**Dream AI:** BOLT

---

## Personal / Lifestyle

### 79. Smart Grocery List from Chat
**Description:** Monitors family WhatsApp/Telegram group for grocery mentions ("we need milk") → adds to shared list → deduplicates → organizes by store aisle.
**Prompt:** "Scan this week's family chat for grocery mentions. Add to shared list. Remove duplicates. Organize by: produce, dairy, meats, pantry."
**Dream AI:** Personal assistant

### 80. Fitness & Nutrition Tracker
**Description:** Logs workouts, meals, sleep. Correlates with energy levels and mood. Sends insights and recommendations.
**Prompt:** "This week's health data: 4 workouts, average 6.5 hours sleep, energy dipped Wednesday-Thursday. Correlate with my schedule — too many meetings on those days?"
**Dream AI:** Health / Personal AI

### 81. Home Maintenance Scheduler
**Description:** Tracks home maintenance tasks by season. Schedules HVAC service, gutter cleaning, filter changes, etc.
**Prompt:** "What home maintenance is due this month? Schedule HVAC tune-up, remind me to replace furnace filter, check smoke detector batteries."
**Dream AI:** Personal assistant

### 82. Personal Finance Dashboard
**Description:** Aggregates bank accounts, investments, credit cards. Daily net worth update, spending alerts, budget tracking.
**Prompt:** "Daily financial snapshot: net worth, today's spending, upcoming bills this week, any unusual transactions. Alert if anything looks off."
**Dream AI:** Personal / Finance

### 83. Gift Finder
**Description:** Researches gift options based on recipient's interests, your budget, and occasion. Generates shortlist with links.
**Prompt:** "Find birthday gifts for my dad: loves golf, grilling, and whiskey. Budget $100-200. Something he wouldn't buy himself. Top 5 options with links."
**Dream AI:** Personal assistant

### 84. Automated Plant Care
**Description:** Monitors plant sensor data (moisture, light, temperature). Schedules watering, alerts on issues, seasonal care tips.
**Prompt:** "Check plant sensors: living room pothos soil moisture at 20% (needs water), bedroom snake plant getting too much direct light. Add watering to today's tasks."
**Dream AI:** IoT / Personal

---

## Sales & Marketing (Dream AI specific applications)

### 85. Prospect Cold Email Drafter (NO SEND)
**Description:** Researches prospect → identifies pain points → drafts personalized email → sends to human for approval ONLY.
**Prompt:** "Research [Company]. Find: revenue leaks, tech stack gaps, recent news. Draft a cold email highlighting how AI receptionist could save them $X/month. DO NOT SEND — show me for approval."
**Dream AI:** AXIOM / outreach

### 86. Proposal ROI Calculator
**Description:** Auto-calculates ROI for proposals based on prospect's actual numbers (call volume, miss rate, avg deal size).
**Prompt:** "Client misses 30 calls/month, avg job value $500, 60% answer rate. Calculate annual revenue loss. Show ROI of $497/mo AI solution over 12 months."
**Dream AI:** AXIOM / sales

### 87. Win/Loss Analysis
**Description:** Reviews won and lost deals. Identifies patterns in why deals close or fall through. Improves sales process.
**Prompt:** "Analyze last quarter's deals: what patterns do you see in won vs. lost? Common objections? Pricing sensitivity? Recommend 3 process improvements."
**Dream AI:** METRIC / sales intelligence

### 88. Client Health Score Monitor
**Description:** Tracks usage, engagement, support tickets, and NPS for each client. Alerts on churn risk.
**Prompt:** "Weekly client health report: who's at risk (score <40)? Who's a champion (score >80, potential referral)? Any usage drops this month?"
**Dream AI:** PULSE / client success

### 89. Upsell Opportunity Scanner
**Description:** Analyzes client usage patterns to identify upsell opportunities. When a client hits usage limits or uses features heavily, flags for expansion conversation.
**Prompt:** "Scan client usage. Who's hitting limits on their current tier? Who's using features that suggest they need Pro? Draft upsell proposals for top 5."
**Dream AI:** PULSE / FORGE

---

## Summary: Total Use Case Database

| Source | Count |
|--------|-------|
| awesome-openclaw-usecases (GitHub) | 40 |
| YouTube/community (original) | 30 |
| useclaw.vercel.app / TLDL survey | 44 |
| **Total unique use cases** | **100+** |

## Adoption & Satisfaction Data (from TLDL survey, 100+ users)

| Category | Adoption | Satisfaction |
|----------|----------|-------------|
| Content automation | 35% | 4.5/5 |
| Research & data | 28% | 4.3/5 |
| Email management | 20% | 4.0/5 |
| Coding assistance | 15% | 4.8/5 |

## Key Insight for Dream AI

The highest-satisfaction use cases (coding, 4.8/5) require technical setup. The highest-adoption use cases (content, 35%) are the easiest to sell to non-technical SMEs — exactly Dream AI's target market.

**Dream AI's play:** Sell the content + productivity use cases to SMEs (easy adoption, high satisfaction), then upsell into business operations (CRM, support, invoicing) as they mature.

---

# Use Cases from useclaw.vercel.app (indirect capture)

Source: lablab.ai, vercel docs, community search results. Site is client-rendered, no API available.

---

### 90. WorkFlow AI Assistant (Email + Tasks + Calendar)
**Description:** Reads emails, organizes tasks, creates documents, summarizes meetings, resolves schedule conflicts, balances workloads. Saves 15-20 hrs/week.
**Dream AI:** Productivity AI Employee

### 91. AI Photo Memory Search (Fotopico)
**Description:** AI-powered photo tagging. Natural language search for instant visual memory retrieval. "Find photos from Emma's birthday 2024."
**Dream AI:** Personal / creative vertical

### 92. Knowledge Worker Automation (IBM watsonx)
**Description:** Handles emails, calendars, reports, admin tasks for knowledge workers. Adapts to individual style. Saves 15-20 hours weekly.
**Dream AI:** Executive AI Employee

### 93. B2B Dispute Mediation (Veredict)
**Description:** Automates dispute mediation by auditing WhatsApp/CRM context. Releases on-chain payments intelligently based on evidence.
**Dream AI:** Legal/contract vertical

### 94. AI Financial Automation for SMBs
**Description:** Classifies expenses, generates invoices, detects fraud, forecasts cashflow. Saves 42 hours/month at 95%+ accuracy.
**Dream AI:** Finance AI Employee / FORGE

### 95. Tariff/Customs Classification (TariffAgent)
**Description:** Classifies invoices (PDF/photo/text) → generates HS codes → calculates duties → produces customs PDFs. Saves thousands per shipment.
**Dream AI:** Logistics/trade vertical

### 96. Credit Risk Automation (CreditNexus)
**Description:** Automated credit risk assessment with data retrieval, ML scoring, IFRS-9 compliance. Reduces decisions from weeks to milliseconds.
**Dream AI:** Finance vertical

### 97. AI Real Estate Marketplace Agent
**Description:** Smart property recommendations based on user needs. Ensures transparency and fraud resistance. Matches buyers to properties.
**Dream AI:** Real Estate AI Playbook feature

### 98. Log Anomaly Detection (Log Sentinel)
**Description:** Detects log anomalies, groups incidents, generates schema-validated explanations using local LLMs. Proactive infrastructure monitoring.
**Dream AI:** BOLT / DevOps AI Employee

### 99. Fraud Investigation Support (CasePilot)
**Description:** Supports fraud investigations via graph analysis, geospatial checks, AI-assisted reporting with self-improving memory.
**Dream AI:** Legal/compliance vertical

### 100. Intelligent Trading Analyst
**Description:** Explains market moves (RSI/ATR + news), detects risky behaviors, drafts shareable posts for trading communities.
**Dream AI:** Finance vertical

### 101. Brain MRI Analysis (NeuroScan AI)
**Description:** Analyzes brain MRIs with Roboflow and Gemini for rapid diagnoses and treatment recommendations.
**Dream AI:** Healthcare vertical

### 102. Receipt OCR Expense Categorizer
**Description:** Extracts amount, vendor, date from receipts. Suggests categories without manual input. Auto-logs to expense tracker.
**Dream AI:** Finance AI Employee

### 103. Agent Skill Publishing (ClawHub)
**Description:** Public skill registry for publishing, versioning, and searching agent SKILL.md files. Like npm for AI agent capabilities.
**Dream AI:** Internal use — publish Dream AI skills

### 104. Mobile App Idea Generator & Builder
**Description:** Generates mobile app ideas from market gaps → builds prototype → pushes to GitHub → submits to App Store.
**Dream AI:** Catalyst / digital products

### 105. Calendar Conflict Resolution with Workload Balancing
**Description:** Detects schedule conflicts across team → proposes optimal resolutions → balances workload → sends calendar updates.
**Dream AI:** Operations AI Employee

### 106. AI Document Filing & Organization
**Description:** Automatically files documents into correct folders based on content. Renames files. Maintains folder structure.
**Dream AI:** Personal / operations

### 107. WhatsApp Group Intelligence
**Description:** Monitors WhatsApp groups for: customer requests, action items, delivery issues. Routes to appropriate team members.
**Dream AI:** Customer Success AI Employee

### 108. Schedule Conflict → Auto-Reschedule
**Description:** When a meeting conflicts, proposes 3 alternative times, checks all participants' availability, sends reschedule invites.
**Dream AI:** Personal AI Employee

### 109. Expense Forecasting & Cashflow Prediction
**Description:** Analyzes historical expenses → predicts future cashflow → flags potential shortfalls → suggests cost optimizations.
**Dream AI:** Finance AI Employee

### 110. HS Code Auto-Classifier
**Description:** Reads product descriptions → assigns correct 10-digit HS codes → calculates applicable duties → generates customs documentation.
**Dream AI:** Trade/logistics vertical

---

## Final Database Count

| Source | Use Cases |
|--------|-----------|
| awesome-openclaw-usecases (GitHub) | 40 |
| YouTube/community (original batch) | 30 |
| TLDL survey + community sources | 44 |
| useclaw.vercel.app (indirect) | 21 |
| **TOTAL** | **135** |

