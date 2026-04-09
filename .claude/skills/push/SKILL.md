---
name: push
description: >
  Exports approved outreach emails into the user's preferred format: sequencer
  CSV, plain text, or formatted for specific tools. Triggers: /push, "export
  emails", "push to instantly", "push to outreach". Reads: ./gtm/prospects.md,
  ./gtm/icp.md (for stack info). Writes: output files in ./gtm/export/.
  Standalone: yes.
allowed-tools:
  - Read
  - Write
  - Edit
  - Grep
  - Glob
  - Bash
---

# /push — Export to Sequencer or File

You take approved outreach emails and format them for the user's email
sequencer, or export them as plain text files ready to copy-paste.

---

## Process

### Step 1: Detect Format + Connected Tools

Check `./gtm/icp.md` for Connected Tools and Stack sections.

**If tools are connected, offer direct push first:**

```
  {If Instantly API connected:}
  I can push these directly to Instantly as a new
  campaign. Or export as files. Which do you prefer?

  {If HubSpot MCP connected:}
  I can create these as HubSpot contacts with the
  email sequence attached. Or export as files.

  {If Clay MCP connected:}
  I can push these contacts to a Clay table for
  further enrichment before sending.
```

**If no direct push tools, offer file formats:**

```
  How do you want these exported?

  1. Plain text (one file, copy-paste ready)
  2. CSV (for Instantly, Apollo, Smartlead, etc.)
  3. Outreach/Salesloft format
  4. HubSpot sequences format
  5. LinkedIn copy-paste (one file with all messages)
  6. LinkedIn CSV (HeyReach, Dripify, etc.)
  7. Both email + LinkedIn files
  8. Just show me — I'll copy them manually
```

### Step 2: Gather Approved Outreach

Read `./gtm/prospects.md` for all prospects with status "outreach-written".

Check what was written — email only, LinkedIn only, or both. Export
whatever exists.

If no outreach has been written, suggest: "Run /outreach first to write
outreach for your prospect list."

### Step 3: Export

#### Plain Text (.txt)

Simple, universal. One file with all emails separated by dividers.

Create `./gtm/export/outreach-{date}.txt`:

```
================================================================
TO: {prospect name} — {title} — {company}
EMAIL: {email if known, otherwise "[find email]"}
LINKEDIN: {url if known}
================================================================

Subject: {subject line}

{email body}

================================================================
{next email...}
```

#### CSV (Instantly, Apollo, Smartlead, etc.)

Create `./gtm/export/outreach-{date}.csv`:

```
first_name,last_name,email,company,title,subject,body,linkedin_url
{first},{last},{email or empty},{company},{title},{subject},{body with newlines as \n},{linkedin}
```

**CSV formatting rules:**
- Escape commas and quotes in body text
- Replace newlines with `\n` in the body column
- Include LinkedIn URL column even if empty
- Email column: use company domain email if found, otherwise leave blank
  with a note that they need to find it

#### Outreach / Salesloft Format

Create `./gtm/export/outreach-{date}-sequences.txt`:

```
SEQUENCE: Signal Outbound — {date}
STEP 1: Email (Day 0)

---

PROSPECT: {name} — {company}
Subject: {subject}
Body:
{email body}

---

{next prospect...}
```

#### HubSpot Sequences Format

Create `./gtm/export/outreach-{date}-hubspot.csv`:

```
Email,First Name,Last Name,Company,Job Title,Contact owner
{email},{first},{last},{company},{title},{user name from voice.md}
```

Plus a separate template file with the email body for the sequence step.

#### LinkedIn Copy-Paste (.txt)

Create `./gtm/export/linkedin-{date}.txt`:

```
================================================================
TO: {prospect name} — {title} — {company}
LINKEDIN: {linkedin URL}
CHARS: {character count}
================================================================

{linkedin connection request message}

================================================================
{next prospect...}
```

#### LinkedIn API (HeyReach, Dripify, etc.)

Create `./gtm/export/linkedin-{date}.csv`:

```
linkedin_url,first_name,last_name,company,title,message
{url},{first},{last},{company},{title},{message}
```

**CSV formatting for LinkedIn messages:**
- Escape commas and quotes in message text
- Replace newlines with spaces (LinkedIn messages are single-block)
- Verify every message is under 300 characters in the CSV

If the user has a LinkedIn automation tool in their stack (from icp.md),
format for that specific tool. If not, default to the generic CSV that
most tools can import.

#### Direct Push: Instantly API

If `$INSTANTLY_API_KEY` is set, push directly:

```bash
# Create campaign
curl -s "https://api.instantly.ai/api/v1/campaign/create" \
  -H "Content-Type: application/json" \
  -d '{
    "api_key": "'$INSTANTLY_API_KEY'",
    "name": "Signal Outbound — {date}",
    "schedule": {...}
  }'

# Add leads to campaign
curl -s "https://api.instantly.ai/api/v1/lead/add" \
  -H "Content-Type: application/json" \
  -d '{
    "api_key": "'$INSTANTLY_API_KEY'",
    "campaign_id": "{id from create}",
    "leads": [
      {
        "email": "{email}",
        "first_name": "{first}",
        "last_name": "{last}",
        "company_name": "{company}",
        "custom_variables": {
          "signal": "{signal}",
          "subject": "{subject}",
          "body": "{body}"
        }
      }
    ]
  }'
```

**NEVER hardcode the API key.** Always use $INSTANTLY_API_KEY.

After push, show:
```
  Pushed {N} leads to Instantly campaign
  "{campaign name}". Check your Instantly
  dashboard to review and activate.
```

#### Direct Push: Clay MCP

If Clay MCP is connected, offer to push contacts to a Clay table:

```
mcp__claude_ai_Clay__track-event:
  - event_name: "outreach_ready"
  - properties: {prospect data + outreach copy}
```

Or use Clay's account management:
```
mcp__claude_ai_Clay__add-contact-data-points:
  - Add signal, outreach copy, and status to existing contacts
```

### Step 4: Present Export

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  EXPORT COMPLETE
  {date}

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  Format: {format name}
  Prospects: {N}
  File: ./gtm/export/{filename}

  ──────────────────────────────────────────────

  MISSING EMAILS ({N} of {total} need emails)
  {If any prospects have "needs enrichment":}

  These prospects need email addresses. Options:

  {If API key available but wasn't used:}
  → I can look these up now with {provider}.
    Say "enrich" and I'll find the missing emails.

  {If API key mentioned but not connected:}
  → Connect your {provider} API key to find them:
    export {PROVIDER}_API_KEY=your_key_here
    Then say "enrich" and I'll fill them in.

  {If no enrichment tools:}
  → Find them in Apollo, Hunter, or ZoomInfo
  → Search LinkedIn profiles for contact info
  → Try {company domain} patterns (first.last@,
    flast@, firstinitiallast@)

  ──────────────────────────────────────────────

  DELIVERABILITY CHECK
  ─────────────────────────────────────────────

  Before sending, make sure:

  ✓ Domain authentication (SPF, DKIM, DMARC)
    set up on your sending domain
  ✓ Sending domain is warmed up (if new, start
    with 5-10/day and ramp over 2-3 weeks)
  ✓ Daily send limit: stay under 50/day per
    mailbox for cold outreach
  ✓ Don't send all {N} emails at once — spread
    across 2-3 days minimum

  If you're not sure about any of these, your
  sequencer (Instantly, Apollo, etc.) usually
  handles warmup and send pacing for you.

  ──────────────────────────────────────────────

  NEXT STEPS

  1. {If emails missing:} Find missing email
     addresses using your email finder
  2. Check deliverability (domain auth, warmup)
  3. Upload {filename} to {sequencer name}
  4. Spread sends across 2-3 days (max 50/day)
  5. Come back after replies and run /outreach
     to log what worked (builds the learning loop)

  ──────────────────────────────────────────────

  After you've sent these, tell me what got
  replies. I'll log it and adjust future outreach.
```

### Step 5: Update Status

Update prospects.md: change status from "outreach-written" to "sent"
for all exported prospects.

---

## Handling Replies (Post-Send)

When the user comes back to log results, make it structured and easy.

### Structured Result Logging

Show the sent prospect list and ask for results:

```
  RESULTS LOG — Batch {date}
  ─────────────────────────────────────────────

  {N} prospects sent. Mark each one:

  1. {Name} at {Company} ({channel})
  2. {Name} at {Company} ({channel})
  3. {Name} at {Company} ({channel})
  ...

  Tell me which ones replied. Examples:
  - "1 and 4 replied"
  - "got a reply from {name}"
  - "3 replies out of 10"
  - "nobody replied"

  Any details on what they said helps me
  write better next time.
```

The user can respond however they want — structured or casual:

- "1 and 4 replied" → mark those as replied, rest as no-reply
- "Got a reply from {name}" → update that one
- "3 replies, don't remember which" → ask who, or mark batch rate
- "Nobody replied" → mark all as no-reply, still valuable data
- "Got a LinkedIn accept from {name}" → update status + channel

**learnings.md format — DETAILED (this drives self-improvement):**

```markdown
## {date} — Batch: {N} sent, {N} replied

### Summary
- Emails sent: {N} | Replies: {N} | Reply rate: {X}%
- LinkedIn sent: {N} | Accepts: {N} | Accept rate: {X}%
- Best signal type this batch: {type}
- Best opener type this batch: {type}

### Replies
- {Name} at {Company}
  Channel: {email / linkedin}
  Signal type: {proven / high-intent / moderate / social}
  Signal used: {specific signal}
  Opener type: {observation / signal / question / buyer / social}
  Bridge angle: {methodology / scale / accuracy / api / price / workflow}
  CTA type: {send-address / quick-call / side-by-side / etc.}
  Notes: {what they said, if shared}

### No Reply
- {Name} at {Company}
  Channel: {email / linkedin}
  Signal type: {type}
  Opener type: {type}
  Bridge angle: {angle}
  CTA type: {type}

### Patterns
- Signal: {which signal types got replies vs didn't}
- Opener: {which opener types performed}
- Bridge: {which bridge angles worked}
- CTA: {which CTAs got responses}
- Channel: {email vs LinkedIn performance}
- Title: {which buyer titles responded — VP vs Director etc.}

### Cumulative Stats (update across all batches)
- Total sent: {N} | Total replies: {N} | Overall rate: {X}%
- Best performing combination: {opener} + {bridge} + {CTA}
- Worst performing combination: {what to avoid}
```

After logging, show the user:

```
  LEARNING UPDATE
  ─────────────────────────────────────────────

  Logged: {N} replies, {N} no-replies
  Reply rate this batch: {X}%
  Cumulative reply rate: {X}% across {N} batches

  {if patterns found:}
  WHAT'S WORKING
  • {signal type}: {X}% reply rate ({N} sends)
  • {opener type}: {X}% reply rate
  • {bridge angle}: {X}% reply rate
  • {channel}: {X}% reply rate

  {if anti-patterns found:}
  WHAT'S NOT
  • {signal/opener/bridge}: 0 replies across {N} sends
  • Deprioritizing for next batch.

  {Show progression:}
  BATCH PROGRESSION
  Batch 1: {N} sent → {N} replied ({X}%)
  Batch 2: {N} sent → {N} replied ({X}%)
  ...
  Trend: {improving / flat / declining}

  ──────────────────────────────────────────────

  Ready for the next batch. I'll find new
  companies showing fresh signals and apply
  what worked.

  Say "go" to start the next batch.
```

### Auto-Continue to Next Batch

**After logging results, offer to start the next batch immediately.**
If the user says "go", "next batch", "keep going", or similar:

1. Auto-continue to /prospect (which dedupes against existing)
2. /prospect will use updated learnings to inform signal prioritization
3. /outreach will use updated learnings to weight winning patterns

This is the core loop: send → log → learn → find → write → send.
Each cycle should be faster and more effective than the last.

---

## Edge Cases

**No sequencer in stack:**
Default to plain text. Mention that a sequencer like Instantly or
Apollo can automate sending at scale.

**User wants to send directly from here:**
This kit doesn't send emails. It writes and exports them. The user
sends from their own tool. This is by design — keeps the kit simple
and avoids needing email API credentials.

**User wants follow-up sequences:**
First-touch only for now. Suggest they come back in 3-5 days, check
replies, and run another batch for non-responders with a different
angle.
