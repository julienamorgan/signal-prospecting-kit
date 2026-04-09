# Signal Prospecting Kit

An orchestrated outbound system for Claude Code. Six skills that chain together to profile your market, find buying signals, discover prospects, and write personalized cold emails and LinkedIn messages. Not a flat collection of prompts — a system with shared context that gets smarter with every send.

## Install

```bash
git clone https://github.com/julienamorgan/signal-prospecting-kit.git
cd signal-prospecting-kit
claude
```

That's it. The skills live in `.claude/skills/` inside the repo, so Claude Code picks them up automatically. No copying files, no install script, no restart. `/start`, `/profile`, `/signal-scout`, `/prospect`, `/outreach`, and `/push` are available immediately.

## Quick Start

```
/start          → Company, ICP, stack, past deals, voice
/profile        → Competitor research + positioning (you approve)
/signal-scout   → Buying signals for your market (you approve)
/prospect       → Companies + contacts showing signals (you approve)
/outreach       → Cold emails + LinkedIn for each prospect (you approve)
/push           → Export to sequencer, LinkedIn tool, or copy-paste
```

Each stage has an approval gate. Nothing moves forward without your confirmation. After `/start`, the system auto-continues through each stage — you just confirm at each gate.

## How It Works

**`/start`** asks two things: your business setup (domain, ICP, stack, API keys, existing customers to exclude) and examples of outreach that worked plus deals you've won. It researches your company and builds the foundation.

**`/profile`** researches your competitors, maps the landscape, finds your white space, and refines your ICP with positioning angles and disqualifiers.

**`/signal-scout`** identifies which buying signals matter for your market. If you provided won-deal examples, it starts with proven signals — events that actually preceded your closed deals. Then adds theoretical signals from research, including social signals (Reddit threads, X posts, LinkedIn engagement). You confirm the signal list.

**`/prospect`** searches four channels for companies showing those signals: behavioral signals (LinkedIn posts, G2 reviews), social community mining (Reddit, X), company events, and proxy signals. It scores on fit + timing + social warmth, finds the right contact at each company, and enriches emails using whatever tools you have connected — Clay, Apollo, Hunter, or web research as a fallback. Existing customers and past contacts are automatically excluded.

**`/outreach`** asks your channel preference (email, LinkedIn, or both), then writes personalized outreach for each prospect. Emails use a 4-paragraph framework with rotating openers, bridges, and CTAs — no two emails in a batch read the same. LinkedIn messages are under 300 characters with non-obvious signals. Everything passes through a humanizer before you see it.

**`/push`** pushes directly to your tools if they're connected — Instantly API creates campaigns automatically, Clay MCP syncs contacts. Otherwise exports as CSV, formatted files, or plain text for copy-paste. Supports Instantly, Apollo, Outreach, HubSpot, HeyReach, Dripify, and generic CSV.

## Tool Connections

The kit works with zero tools connected — it uses web research for everything. But it gets significantly better when you connect your existing tools:

| Tool | How to Connect | What It Enables |
|------|---------------|----------------|
| Clay | Install Clay MCP server | Contact enrichment with emails in one step, company data, custom workflows |
| Apollo | `export APOLLO_API_KEY=your_key` | Email finding by name + company |
| Hunter | `export HUNTER_API_KEY=your_key` | Email finding by domain |
| Instantly | `export INSTANTLY_API_KEY=your_key` | Direct campaign creation — no CSV upload needed |

The system detects connected tools automatically during `/start` and uses the best available option at each step. You can connect tools at any time — run `/start` again and it picks them up.

## The Learning Loop

After you send outreach, come back and tell the system what got replies. It shows your sent list and asks you to mark who replied — just say "1 and 4 replied" or "nobody replied." It tracks which signal types, opener patterns, bridge angles, CTAs, and channels performed. Each batch gets smarter:

- **Batch 1:** Baseline — uses won-deal patterns and default rotation
- **Batch 2-3:** Starts weighting toward what worked
- **Batch 4+:** Clear patterns emerge — the system tells you what's working and applies it automatically

This is the compound advantage. Every other skill kit starts from zero every time.

## The System Behind It

Signal-based outbound works on two axes: fit and timing.

Fit is your ICP — does this company match the profile of companies you've closed before? Timing is buying signals — is something happening right now that makes them more likely to act?

Most teams score on fit alone. The ones that add timing get to prospects before competitors know there's an opportunity. This kit builds both axes and writes outreach that references something real and recent.

## What You Can Expect

First setup takes about 30 minutes. After that, each batch of 10-15 prospects takes 15-20 minutes. The system gets faster as your signals and ICP are dialed in.

Built on the same methodology behind:
- Scoring 2,065 accounts and finding 435 worth calling
- 8.26% cold email reply rate (industry average: 1-2%)
- 700 net-new prospects surfaced for a single client

## Want Help Setting This Up?

This kit gives you the methodology. If you want hands-on help — buying signal detection wired into your CRM, automated scoring, daily prospect discovery, your team trained to run it — that's what I do as a fractional GTM engineer.

[climategrowthadvisors.com](https://climategrowthadvisors.com)
