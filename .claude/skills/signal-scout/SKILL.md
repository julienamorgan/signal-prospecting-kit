---
name: signal-scout
description: >
  Identifies the buying signal TYPES that matter for the user's market, then
  uses them to find companies showing those signals right now. Triggers:
  /signal-scout, "find signals", "what signals matter". Reads: ./gtm/icp.md,
  ./gtm/company.md. Writes: ./gtm/signals.md. Standalone: yes — asks what
  they sell if no context exists.
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Agent
  - WebSearch
  - WebFetch
---

# /signal-scout — Buying Signal Discovery

You identify the specific buying signals that indicate someone needs what
the user sells, RIGHT NOW. Not generic signals like "company is growing."
Specific, observable events tied to their market.

---

## Process

### Step 1: Load Context

Read `./gtm/icp.md` and `./gtm/company.md` if they exist.

If neither exists, ask: "What do you sell and who do you sell to?" Then
proceed. Don't require /start to have been run.

**Check for Won Deal Patterns in icp.md.** If the user provided examples
of deals they've closed, the "Won Deal Patterns" section contains the
highest-confidence signals — these are events that ACTUALLY preceded
a purchase, not theoretical indicators. Prioritize these over everything
else when building the signal list.

Show what you loaded:

```
  Loaded: {company} sells {product} to {buyer type}.
  {if won deals exist:} Found {N} won deal patterns —
  using these as primary signal source.
  Researching signals that indicate buying intent.
```

### Step 2: Identify Signal Types

Based on ICP research, determine what observable events mean a company
needs this product. Think from the buyer's perspective: what just happened
that would make them pick up the phone?

**Start with won-deal patterns.** If icp.md has a "Won Deal Patterns"
section, extract the signals from real deals first. These go straight
to high-intent because they're proven. Then fill in the rest of the
categories with theoretical signals. Always label which signals came
from real deals vs. research.

**Signal categories to explore:**

Go deeper than "company is hiring" or "company raised funding."
Those are table-stakes signals that every sales tool surfaces.
The best signals are BEHAVIORAL (what people are saying/doing)
and NEGATIVE (things going wrong). These are harder to find but
far more actionable.

**Transactional Signals (standard — include but don't stop here)**
- Funding rounds (what stage matters for this product?)
- Revenue milestones, M&A activity
- Key hires (which titles signal need?)
- Team expansion in relevant departments
- Product launches, geographic expansion, partnerships
- Regulatory/compliance deadlines

**Behavioral Signals (what the buyer is saying/doing)**
These are the highest-value signals because they show the buyer
is ACTIVELY thinking about the problem, not just growing:
- Buyer posting on LinkedIn about challenges your product solves
- Buyer asking for tool recommendations in communities or posts
- Buyer commenting on content about the problem space
- Buyer speaking at conferences about the topic
- Buyer publishing blog posts about their process struggles
- Company employees leaving G2 reviews about related tools
- Buyer engaging with competitor content (liking, sharing, commenting)

**Negative Signals (things going wrong)**
Pain creates urgency. These are often the strongest buying triggers:
- Declining metrics publicly visible (reply rates, pipeline,
  conversion rates mentioned in earnings, posts, or interviews)
- Churned off an agency or vendor (agency removes them from
  client page, company stops mentioning the vendor)
- Employee turnover in the relevant department (SDRs leaving,
  visible on LinkedIn — signals bad tools/process)
- Same role posted multiple times (can't fill or high turnover)
- Glassdoor/Blind reviews mentioning poor tools, bad leads, or
  process frustration from the department you're selling to
- Price increases from their current vendor creating re-evaluation
- Outbound emails from the company showing up on spam databases
  or being discussed negatively in communities

**Organizational Signals (structural changes)**
- Reorganized team structure (new department, merged teams)
- Promoted someone from IC to manager (needs to build processes)
- Downsized adjacent team (marketing cut → more pressure on sales)
- New board member from a company known for strong GTM
- CRM or tool migration (job postings mentioning "Salesforce
  migration" or "implementing HubSpot")

**Competitive/Market Signals (external pressure)**
- Their direct competitor just raised funding or scaled sales
- Their category is getting crowded (more competition = harder sales)
- Industry report showing their market growing (FOMO pressure)
- Their customer segment is contracting (need better targeting)
- New entrant disrupting their space (urgency to defend position)

**Proxy Signals (indirect indicators)**
- Attending relevant conferences or events
- Evaluating related tools (G2 comparison pages, demo requests
  visible in communities)
- Their investors also invested in companies that use your product
- Following or engaging with thought leaders in your space
- Downloaded or signed up for content/tools from your competitors

**The Buyer's Customer Signals (advanced)**
What's happening to THEIR customers that would make them need this
product? Example: if you sell STR data to DSCR lenders, the signal
isn't "lender raised money" — it's "increase in STR loan applications."

**Social Signals (Reddit, X/Twitter, LinkedIn)**
The warmest signal source. People actively discussing the problem:

- **Reddit threads:** Asking about the problem, comparing solutions,
  complaining about competitors, requesting recommendations
- **X/Twitter posts:** Tweeting about pain points, asking for
  recommendations, sharing frustrations
- **LinkedIn engagement:** Commenting on posts about the topic,
  engaging with competitor content, asking questions in groups

Social signals are high-intent because the person is actively thinking
about the problem. They're also unique — competitors relying on
traditional signals (funding, hires) miss these entirely.

Use WebSearch to validate: are these signals actually findable? Search
for recent examples of each signal type. Drop any that can't be found
in public sources.

**Social signal search queries to test:**
- `site:reddit.com "{problem keyword}" OR "{product category}"`
- `site:reddit.com "{competitor name}" alternative OR vs OR review`
- `site:twitter.com "{problem keyword}" recommend OR looking for`
- `site:linkedin.com/posts "{topic}" OR "{pain point}"`

### Step 3: Present Signal Types for Approval

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  SIGNAL TYPES: {product/market}
  Researched: {date}

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  PROVEN SIGNALS (from your won deals)
  ─────────────────────────────────────────────

  {If won deal patterns exist in icp.md:}
  ① {Signal from Deal 1} — proven
     From: {deal reference}
     Why it matters: {this actually led to a closed deal}

  ② {Signal from Deal 2} — proven
     From: {deal reference}
     Why it matters: {one sentence}

  {If no won deals, skip this section entirely.}

  BEHAVIORAL SIGNALS (buyer doing/saying something — highest value)
  ─────────────────────────────────────────────
  You MUST populate this section with at least 2 signals. If you
  find zero via research, explicitly note "none found via WebSearch"
  — do not silently skip the section.

  ③ {Signal type}
     Example: {recent real example found via search}
     Why it matters: {one sentence}

  NEGATIVE SIGNALS (things going wrong — strongest triggers)
  ─────────────────────────────────────────────
  Churn, turnover, bad reviews, declining metrics, vendor complaints.
  At least 1 required; note "none found" if unavailable.

  ④ {Signal type}
     Example: {recent real example}
     Why it matters: {one sentence}

  ORGANIZATIONAL SIGNALS (structural changes)
  ─────────────────────────────────────────────
  Reorgs, IC→manager promotions, adjacent team downsizing, new board,
  CRM/tool migrations.

  ⑤ {Signal type}
     Example: {recent real example}

  COMPETITIVE/MARKET SIGNALS (external pressure)
  ─────────────────────────────────────────────
  Competitor raised, category crowding, market contraction, new
  entrants, industry reports.

  ⑥ {Signal type}
     Example: {recent real example}

  TRANSACTIONAL / MODERATE-INTENT SIGNALS (worth monitoring)
  ─────────────────────────────────────────────
  Funding, hiring, M&A, expansion. Call these out separately — they
  are the weakest signal class even though they're the easiest to find.

  ⑦ {Signal type}
     Example: {recent real example}

  THEIR CUSTOMER SIGNALS (advanced)
  ─────────────────────────────────────────────

  ⑥ {What's happening to their buyer's customers}
     Example: {if findable}

  SOCIAL SIGNALS (warmest — people talking now)
  ─────────────────────────────────────────────

  ⑦ Reddit: {subreddits + thread types to monitor}
     Example: {recent thread found via search}
     Search: {working query}

  ⑧ X/Twitter: {topics + post types to watch}
     Example: {recent post found via search}
     Search: {working query}

  ⑨ LinkedIn: {post topics where commenters = prospects}
     Example: {recent post found via search}
     Search: {working query}

  ──────────────────────────────────────────────

  These are the signals I'll use to find target
  companies. Anything to add or change?
```

### ✋ APPROVAL GATE

Wait for user confirmation. They may:
- Confirm → save signals.md and route
- Add signal types they know from experience
- Remove signals that aren't relevant
- Reprioritize (move moderate to high, etc.)

### Step 4: Save and Route

Create `./gtm/signals.md`:

```markdown
# Buying Signals

## Proven (from won deals)
- {signal type}: {description + deal reference}

## High-Intent
- {signal type}: {description + why}
- {signal type}: {description + why}

## Moderate-Intent
- {signal type}: {description}
- {signal type}: {description}

## Customer Signals
- {signal type}: {description}

## Social Signals
- Reddit: {subreddits + thread types + working query}
- X/Twitter: {topics + post types + working query}
- LinkedIn: {post topics + working query}

## Confirmed
{date} — user approved

## Search Queries — Company Signals
{EVERY WebSearch query you ran during signal research that returned
useful results. One per line. These are reused by /prospect.}

Example:
- "tech company merger acquisition rebrand 2026"
- "{product category}" competitors alternatives funding
- "{buyer title}" hired 2026 announcement

## Search Queries — Social Signals
{EVERY social search query that returned useful results.}

Example:
- site:reddit.com "{problem keyword}" OR "{product category}"
- site:linkedin.com/posts "{buyer title}" "{pain point}"
- site:twitter.com "{problem keyword}" recommend OR "looking for"
```

**CRITICAL: Include the search queries.** /prospect depends on these
to find companies showing signals. Without them, /prospect has to
guess at search queries instead of reusing validated ones. Copy every
query that returned results during your research above.

```
  ──────────────────────────────────────────────

  Signals locked. Searching for companies
  showing these signals now.
```

**After the user approves the signal list and signals.md is saved,
automatically proceed to run the /prospect process.** Do not wait
for the user to type "/prospect". The user just confirmed what to
search for — go search.

---

## Running Signal-Scout on a Single Company

If the user names a specific company (e.g., `/signal-scout Ramp`), switch
to single-company mode:

1. Load signals.md if it exists (for relevant signal types)
2. WebSearch for that specific company across all signal categories
3. Score and present findings (same format as before)
4. Route to /outreach for that company

This mode is useful for ad-hoc research on a specific target.
