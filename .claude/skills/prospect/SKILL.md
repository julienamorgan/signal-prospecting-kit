---
name: prospect
description: >
  Finds companies matching the ICP that are showing buying signals right now,
  then finds contacts at the best ones. Builds a scored prospect list for
  outreach. Triggers: /prospect, "find companies", "build a list", "find
  prospects". Reads: ./gtm/icp.md, ./gtm/signals.md, ./gtm/company.md.
  Writes: ./gtm/prospects.md. Standalone: yes.
allowed-tools:
  - Read
  - Write
  - Edit
  - Grep
  - Glob
  - Bash
  - Agent
  - WebSearch
  - WebFetch
---

# /prospect — Company + Contact Discovery

You find companies that match the user's ICP and are showing buying signals
right now, then find the right contacts at those companies. You build a
scored, researched prospect list ready for outreach.

---

## Process

### Step 1: Load Context

Read:
- `./gtm/icp.md` — who to target, disqualifiers, buyer situation,
  do-not-contact list, API keys available
- `./gtm/signals.md` — what buying signals to look for (with search queries)
- `./gtm/company.md` — to avoid targeting competitors or the user's own company
- `./gtm/prospects.md` — to avoid duplicates if previous runs exist

If signals.md doesn't exist, ask what signals matter or suggest running
/signal-scout first. Don't guess at signals.

**Load the Do Not Contact list from icp.md.** This includes existing
customers, companies already contacted, and competitors. Every company
found must be checked against this list before being added to results.

**Load Connected Tools from icp.md.** Check what's available:
- Clay (MCP): Can enrich companies, find contacts with emails, run
  subroutines. If connected, use it as primary enrichment in Step 5.
- Apollo (API): Email enrichment. Secondary to Clay if both available.
- Hunter (API): Email finding by name + domain.
- Other tools: Note for export in /push.

**FALLBACK — If Connected Tools section is empty or missing in icp.md,
detect tools yourself before proceeding.** This covers cases where
/start was skipped or tool detection didn't run:
1. Check for MCP tools: look for available tools starting with `mcp__`
   containing `Clay`, `apollo`, `hubspot`, or `instantly`
2. Check env vars with Bash: `[ -n "$APOLLO_API_KEY" ] && echo set`
   (same for HUNTER_API_KEY, INSTANTLY_API_KEY)
3. Use whatever you find. Update icp.md Connected Tools section.

The tool tier determines how Step 5 works:
- **Tier 1 (Clay MCP connected):** Use Clay to find and enrich contacts
  with emails in one step. Most reliable.
- **Tier 2 (API keys available):** Use Apollo/Hunter APIs for email
  enrichment after finding contacts via WebSearch.
- **Tier 3 (no tools):** WebSearch for contacts, mark emails as
  "needs enrichment." Still works, just needs manual email finding.

**You MUST attempt email enrichment in Step 5.** Do not skip it.
Every prospect should have either an email address or an explicit
"needs enrichment" tag with a suggestion to connect Clay or Apollo.

Show what you loaded:

```
  Loaded: ICP ({buyer type}), {N} signal types,
  {N} existing prospects (dedup check active),
  {N} companies on do-not-contact list.
  Email enrichment: {Tier 1: Clay MCP | Tier 2: Apollo/Hunter API | Tier 3: WebSearch only}
  Searching for companies showing buying signals.
```

### Step 2: Find Companies

Use ALL search queries from signals.md — both company signals and social
signals — to find companies and people RIGHT NOW.

**IMPORTANT: Don't default to job boards.** Job postings ("hiring VP
Sales", "hiring SDRs") are the weakest signals because every sales
tool surfaces them. Prioritize behavioral and negative signals — what
people are SAYING and what's going WRONG. These are harder to find
but produce dramatically better prospects.

**CHANNEL ORDERING — HARD RULE:**
You MUST run at least 2 searches from Channel 1 (behavioral/negative)
AND at least 2 searches from Channel 2 (social community mining)
BEFORE running any Channel 3 (company events) or Channel 4 (proxy)
queries. This is not a preference — it is a required execution order.

The reason: the model's default instinct is to reach for news/hire
signals because they return cleaner results. That's exactly the
failure mode this ordering prevents. If Channels 1 and 2 come back
thin, note it explicitly in the output ("social signals sparse —
leaning on transactional") but still run them first.

Track your queries as you go. Before you write any prospect into
the list, verify:
- [ ] ≥2 Channel 1 (behavioral/negative) queries executed
- [ ] ≥2 Channel 2 (social community) queries executed
- [ ] Then, and only then, Channel 3/4 queries

**Channel 1: Behavioral + Negative Signals (highest priority)**
Search for the buyer DOING or SAYING something that reveals pain:

LinkedIn activity searches:
- `site:linkedin.com/posts "{buyer title}" "{problem keyword}"`
  e.g., `"VP Sales" "outbound not working"` or `"Head of Revenue"
  "pipeline challenges"`
- `site:linkedin.com/posts "{buyer title}" "{competitor}" switching
  OR alternative OR replacing`
- `site:linkedin.com/posts "{buyer title}" "looking for" OR
  "anyone recommend" "{product category}"`

G2/review searches:
- `site:g2.com "{competitor}" review negative OR disappointed OR
  switching OR expensive`
- `"{competitor}" churned OR cancelled OR replaced OR "switched to"`

Glassdoor/employee sentiment:
- `site:glassdoor.com "{company}" "sales" OR "SDR" poor tools OR
  bad leads OR no process`
- Look for patterns: SDR turnover, mentions of bad data/tools

Negative event searches:
- `"{company}" layoffs OR restructuring sales team 2026`
- `"{company}" "reply rates" OR "pipeline" declining OR struggling`
- `"{company}" "agency" OR "vendor" churn OR switch OR replace`

What to extract:
- **The person posting/reviewing** = potential buyer
- **The company mentioned** = company with the pain
- **The specific complaint** = use this in the outreach opener

**Channel 2: Social Community Mining**
Find people actively discussing the problem in communities:

Reddit:
- `site:reddit.com "{problem keyword}" OR "{product category}"`
- `site:reddit.com "{competitor}" alternative OR review OR vs`
- `site:reddit.com "{industry}" "looking for" OR "anyone use"`
- Read promising threads via WebFetch — extract company names,
  pain points, identifiable users

X/Twitter:
- `site:twitter.com "{problem keyword}" recommend OR "looking for"`
- `site:twitter.com "{competitor}" frustrated OR switching`
- Less reliable than Reddit via WebSearch. Try it, but don't
  depend on it.

LinkedIn post engagement:
- `site:linkedin.com/posts "{topic}" OR "{pain point}"`
- Commenters are self-identified interested parties (warmest leads)
- Post authors are potential referral sources

**Channel 3: Company Event Signals**
These are standard but still useful — just don't ONLY search here:

Organizational changes:
- `"{company}" new "VP Sales" OR "CRO" OR "Head of Revenue" hired
  OR joined OR appointed 2026`
- `"{company}" "RevOps" OR "Revenue Operations" first hire`
- `"{company}" reorganized OR restructured sales`

Financial events:
- Funding rounds, revenue milestones, M&A
- But go DEEPER: `"{company}" post-funding sales hiring scaling`

Competitive pressure:
- `"{company}" competitor raised OR launched OR expanded`
- `"{industry}" new entrant OR disruption 2026`

**Channel 4: Proxy + Content Signals**
Indirect indicators that someone is thinking about the problem:

- `"{buyer title}" "{conference}" OR "{event}" speaking OR attending`
- `"{company}" blog "sales process" OR "pipeline" OR "outbound"`
- `"{company}" evaluated OR piloting "{related tool}"`
- Companies whose competitors recently scaled their sales teams

**For each company found, capture the SPECIFIC signal:**
Don't just note "hiring VP Sales." Note "VP Sales posted on LinkedIn
Mar 3 about pipeline being 40% below target and asking for tool
recommendations." The specificity is what makes the outreach work.

**For each company found (from any channel):**
1. Verify they match ICP (industry, size, buyer title exists)
2. Check they're not on the disqualifier list from icp.md
3. **Dedup within this run** — if the same company appeared in Reddit

**For each company found (from any channel):**
1. Verify they match ICP (industry, size, buyer title exists)
2. Check they're not on the disqualifier list from icp.md
3. **Dedup within this run** — if the same company appeared in Reddit
   AND news AND LinkedIn, merge into one entry. Note all signal sources.
   Multiple channels = stronger signal (bonus in scoring).
4. Check they're not already in prospects.md (cross-run dedup)
5. Check they're not the user's own company or a competitor
6. Note which signals they're showing, when, and from which channel

**Target: 15-25 unique companies per search run.** Quality over quantity.

**Channel reliability notes:**
- Reddit (`site:reddit.com`) — most reliable social search. Rich threads
  with identifiable companies and pain points.
- LinkedIn (`site:linkedin.com/posts`) — hit or miss via WebSearch. Works
  best for recent, high-engagement posts. May return limited results.
- X/Twitter (`site:twitter.com`) — least reliable via WebSearch. Google
  often returns poor results for Twitter. Try it, but don't depend on
  it as a primary source. If results are thin, skip and spend the time
  on Reddit and LinkedIn instead.

### Step 3: Score and Rank

Score each company on two axes:

**Fit Score (does this company match ICP?)**
- Buyer title exists at this company: +2
- Industry match: +2
- Company size/stage match: +2
- Not a disqualifier: +1 (0 if disqualified)

**Signal Score (is the timing right?)**

Behavioral signals (buyer doing/saying something):
- Buyer posted about the problem on LinkedIn: +4
- Negative G2/Glassdoor review about current solution: +3
- Churned off a competitor or agency: +4
- Employee turnover in relevant department: +3
- Buyer asking for recommendations in community: +4

Standard signals (company events):
- High-intent signal in last 30 days: +3
- High-intent signal in last 90 days: +2
- Moderate signal in last 30 days: +1

Bonuses:
- Multiple signals from different categories: +1
- Social-sourced (Reddit/X/LinkedIn thread): +2
- Buyer's customer signal detected: +1

**Behavioral signals score highest** because the person is actively
thinking about the problem. "VP Sales posted about pipeline struggles"
is worth more than "company raised Series B" because the first shows
awareness, the second just shows budget.

A job posting alone (hiring SDRs, VP Sales) scores as a standard
signal (+2 or +3). A job posting PLUS a behavioral signal (the new
VP posted about needing better tools) scores as behavioral (+4)
plus standard (+2) = much higher.

**Combined Score = Fit + Signal.** Rank by combined score.

Fit max = 7 (title +2, industry +2, size +2, not disqualified +1).
Signal max = 11+ (behavioral +4, standard +3, bonuses +4).
Realistic top score for a strong prospect: 14-18.

**Show the raw score, not a fraction.** Don't write "14/16" or
"14/18" — just write "Score: 14". The scale isn't fixed because
signal bonuses stack.

**Thresholds:**
- Top prospects: 10+
- Worth a look: 6-9
- Dropped: below 6

### Step 4: Present Companies for Approval

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  PROSPECT LIST: {N} companies found
  Searched: {date}

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  TOP PROSPECTS (score 10+)
  ─────────────────────────────────────────────

  ① {Company Name} — {domain} — Score: {X}
     Fit: {industry, size, buyer title exists}
     Signal: {SPECIFIC behavioral event — not "hiring SDRs"
       but "VP Sales posted Mar 3 about pipeline being 40%
       below target" or "3-star G2 review of Clay from their
       SDR manager citing 'no signal layer'"}
     Source: {LinkedIn post / G2 review / Reddit thread /
       Glassdoor / news / job board}
     Why now: {one sentence connecting signal to buying urgency}

  ② {Company Name} — {domain} — Score: {X}
     Fit: {details}
     Signal: {SPECIFIC behavioral event + date}
     Source: {where found}
     Why now: {one sentence}

  ...

  WORTH A LOOK (score 6-9)
  ─────────────────────────────────────────────

  {lower-scored companies, same format}

  DROPPED (disqualified or low-signal)
  ─────────────────────────────────────────────

  {company}: {why dropped — competitor, no signals, etc.}

  ──────────────────────────────────────────────

  Which companies should I find contacts for?
  (Say "all top" or name specific ones.)
```

### ✋ APPROVAL GATE

Wait for user to select companies. They may:
- Say "all top" → research contacts for all top-scored
- Name specific companies → research only those
- Remove some → note why (update disqualifiers if pattern)
- Add companies they know → research those too

### Step 5: Find Contacts + Emails

For each approved company, find the right contact AND their email.

**Target titles** (from icp.md buyer profile):
- Primary: {the decision-maker title from ICP}
- Secondary: {one level up or adjacent title}

**Use the best available tool tier (from Connected Tools in icp.md):**

---

**TIER 1: Clay MCP connected (best)**

Clay's real tool signature (verified against the live MCP — do NOT
invent parameters):

```
mcp__claude_ai_Clay__find-and-enrich-contacts-at-company:
  companyIdentifier: "{domain}"           # REQUIRED. Domain like "stripe.com"
                                          # or LinkedIn URL. Company NAMES alone
                                          # will fail — convert "Stripe" → "stripe.com"
  contactFilters:
    job_title_keywords: ["{title}"]       # array of strings; e.g.
                                          # ["Head of Sustainability", "CSO"]
    # optional additional filters:
    # current_role_max_months_since_start_date: 12  (for new hires)
    # locations: ["United States"]
  dataPoints:
    contactDataPoints:
      - { type: "Email" }                 # request email enrichment
```

There is NO `company_name`, `title_filter`, or `limit` parameter. Use
`companyIdentifier` (domain), `contactFilters.job_title_keywords` (array),
and `dataPoints.contactDataPoints` to request the email.

**Clay is ASYNC — plan for this.** The initial call returns contact
metadata (name, title, LinkedIn URL, company data) synchronously, but
enrichments like `Email` come back with `state: "in-progress", value: null`.
Do not assume the email is in the first response.

Handling async enrichments:
1. Capture the `taskId` returned by the call. Write it to prospects.md
   in the Email column as `pending (clay:{taskId})`.
2. Continue the pipeline — do NOT block on emails for the approval
   gates. The user approves the contact list first.
3. In /push (or just before exporting), retrieve email results by
   calling `mcp__claude_ai_Clay__get-existing-search` with the stored
   taskId. This is the documented polling tool for contact enrichments.
   Poll up to 3-5 times with a short wait between attempts if the
   enrichment state is still `"in-progress"`. Example:

   ```
   mcp__claude_ai_Clay__get-existing-search:
     taskId: "{stored taskId, e.g. mcp-task_abc123}"
   ```

   The response is the same shape as the original find-and-enrich call,
   but with `enrichments[].state` now set to `"completed"` and
   `enrichments[].value` populated with the real email address (or
   `"None Found"` if Clay couldn't find it).
4. If an email is still pending at export time, mark it "needs enrichment"
   and suggest running /push again in a few minutes, or fall back to
   Tier 2 (Apollo/Hunter) for that row.

Multi-contact handling: if Clay returns several people matching the
title filter, pick the highest-ranking person (e.g. CSO > Head of
Sustainability > Sustainability Manager). Only one contact per company.

You can also enrich company data separately if needed:
```
mcp__claude_ai_Clay__find-and-enrich-company:
  companyIdentifier: "{domain}"
  companyDataPoints:
    - { type: "Recent News" }
    - { type: "Headcount Growth" }
```

**Before any Clay call, verify credits** with
`mcp__claude_ai_Clay__get-credits-available`. If
`hasWorkspaceCredits: false`, fall back to Tier 2 or 3 immediately and
tell the user.

**NEVER hardcode or display API keys or credentials.**

If Clay returns no results for a company (empty `contacts` array), fall
back to Tier 2 or 3 for that specific company.

---

**TIER 2: API keys available (Apollo, Hunter, etc.)**

First find the contact name via WebSearch:
- WebSearch: `"{company name}" "{target title}" site:linkedin.com`
- WebSearch: `"{company name}" team OR leadership OR about`
- Company website team/about page via WebFetch

Then enrich email via API (using environment variables):
```bash
# Apollo
curl -s "https://api.apollo.io/v1/people/match" \
  -H "X-Api-Key: $APOLLO_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"name": "...", "organization_name": "..."}'

# Hunter
curl -s "https://api.hunter.io/v2/email-finder?domain={domain}&first_name={first}&last_name={last}&api_key=$HUNTER_API_KEY"
```

**NEVER hardcode or display API keys.** Always use $ENV_VAR.

---

**TIER 3: No tools connected (WebSearch only)**

Find contact via WebSearch:
- WebSearch: `"{company name}" "{target title}" site:linkedin.com`
- WebSearch: `"{company name}" team OR leadership OR about`
- Company website team/about page via WebFetch

For email: note the company's email domain pattern if detectable
(first.last@, flast@). Mark email as "needs enrichment."

Show this once at the end of the contact list:
```
  {N} contacts need email addresses. Options:

  → Connect Clay (richest data, one-step):
    Install the Clay MCP server, then run
    /prospect again.

  → Connect Apollo or Hunter (email finding):
    export APOLLO_API_KEY=your_key_here
    Then run /prospect again.

  → Find manually in Apollo, Hunter, or ZoomInfo.
```

---

**For each contact, capture:**
- Full name (title-case, drop if 1 char or initials only)
- Title
- Company
- LinkedIn URL if found
- Email (from Clay, API, public source, or "needs enrichment")
- Email source (Clay, Apollo, Hunter, public, or pending)

**One contact per company.** The highest-ranking person matching the
buyer title. Don't stack multiple contacts at the same company.

### Step 6: Present Complete Prospect List

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  PROSPECTS READY: {N} contacts at {N} companies
  {date}

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  {Name} — {Title} — {Company}
    Email: {email or "needs enrichment"}
    Signal: {what's happening + date}
    LinkedIn: {URL if found}

  {Name} — {Title} — {Company}
    Email: {email or "needs enrichment"}
    Signal: {what's happening + date}
    LinkedIn: {URL if found}

  ...

  ──────────────────────────────────────────────

  {N} prospects ready for outreach.

  Does this list look right? I'll start writing
  personalized emails once you confirm.
```

### ✋ APPROVAL GATE

Wait for confirmation that the contact list looks right.

**After the user confirms the contact list, automatically proceed to
run the /outreach process.** Do not wait for them to type "/outreach".
Save prospects.md and start writing emails immediately.

### Step 7: Save

Save to `./gtm/prospects.md`:

```markdown
# Prospect List

## Last Updated
{date}

## Batch: {date}

| Name | Title | Company | Domain | Email | Signal | Source | Score | LinkedIn | Status |
|------|-------|---------|--------|-------|--------|--------|-------|----------|--------|
| {name} | {title} | {company} | {domain} | {email} | {signal} | {channel} | {score} | {url} | ready |

## Dropped
- {company}: {reason}
```

Status values: `ready`, `outreach-written`, `sent`, `replied`, `no-reply`

---

## Running Prospect Again

On subsequent runs, /prospect:
- Deduplicates against existing prospects.md
- Shows only new companies not already in the list
- Appends new batch with date header
- Preserves status of existing prospects

---

## Edge Cases

**User provides their own list:**
If the user pastes company names, research each one (signals, contacts)
and add to prospects.md. Skip the search step.

**Not enough results:**
If fewer than 10 companies found, broaden the search:
- Expand to moderate-intent signals
- Lean harder on social channels (Reddit and X often surface
  companies that news searches miss)
- Widen industry or size criteria slightly
- Report what was tried and suggest adjusting ICP

**Social-sourced prospect with no company identified:**
Sometimes a Reddit poster or X user discusses the problem but their
company isn't identifiable. Note the username/handle and the thread
URL. The user can follow up manually or use LinkedIn to identify them.
Add to prospects.md with company as "Unknown — {platform} user" and
include the thread/post URL in the signal column.
