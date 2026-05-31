---
name: ai-hr-weekly-digest
description: Weekly AI & HR Tech digest pipeline — research, build index.html + archive, commit/push to GitHub, draft team email.
---

# Weekly AI & HR Tech Digest — Pipeline Spec

This is the canonical spec. Run it end-to-end on schedule. Every step, tool call, and file op is defined here — ask no questions, just execute.

**Repo:** `/Users/dmcr/Projects/ai-hr-weekly`
**Live index:** https://raw.githubusercontent.com/dmcr85/ai-hr-weekly/main/index.html
**GitHub issues:** https://github.com/dmcr85/ai-hr-weekly/issues

## Dates

- **Working date** = today (digest publication date). Format: `DD Month YYYY` in copy, `YYYY-MM-DD` in filenames.
- **Previous issue date** = today − 7 days. Used for the archive filename.

## Step 1 — Pull team submissions from GitHub

```bash
gh issue list --repo dmcr85/ai-hr-weekly \
  --label poc-idea,investigation-request,feedback \
  --state open --json number,title,body,labels
```

For each open issue: capture `number`, `title`, `body`, `labels`. Include in the digest under **Team Submissions**. If none, render the empty-state div.

After the digest is built AND pushed, for each included issue:

```bash
gh issue comment <number> --repo dmcr85/ai-hr-weekly --body "Included in digest for <DD Month YYYY>"
gh issue close   <number> --repo dmcr85/ai-hr-weekly
```

Only comment/close after the push in Step 6 succeeds — never leave issues closed if the digest failed to ship.

## Step 2 — Research (11 web searches, last 7 days)

Run all 11 searches. Substitute `[month year]` with the current month + year.

**HR & ServiceNow**
1. `ServiceNow HRSD new features store release [month year]`
2. `Now Assist ServiceNow GenAI agentic AI HR [month year]`
3. `Moveworks ServiceNow EmployeeWorks updates [month year]`
4. `Workday AI SAP Joule Microsoft Copilot HR updates [month year]`

**General AI**
5. `Anthropic Claude model release update [month year]`
6. `OpenAI GPT model update news [month year]`
7. `Google Gemini model release update [month year]`
8. `NVIDIA AI GPU announcement [month year]`
9. `DeepSeek model release open source [month year]`
10. `Meta Llama Mistral xAI Grok AI news [month year]`
11. `Any major AI regulatory, acquisition, or infrastructure news`

For each item extract: **headline**, **2–3 sentence summary**, **source URL** (real, from search results — never fabricated), and a **badge** (`NEW` / `HOT` / `BREAKING` / `WATCH` / `Agentic` / `ServiceNow` / `GenAI` etc.).

## Step 3 — Read the current live index.html

Fetch https://raw.githubusercontent.com/dmcr85/ai-hr-weekly/main/index.html. Extract:

1. **Current issue number** → increment by 1 for the new issue.
2. **Full POC Lab tab content** → carry forward verbatim.
3. **Archive tab entries** → keep the 4 most recent; the oldest will be dropped.
4. **Submit Idea tab** → copy verbatim. Never modify.

## Step 4 — Build the new index.html

Single self-contained HTML file. No external CSS, no external JS. Everything inline.

### Header
Issue number, date, "Auto-compiled by Claude on Cowork" badge.

### Tab nav (sticky top, 4 tabs, active tab in green `#00e5a0`)
Latest · POC Lab · Archive · Submit Idea

### Tab 1 — Latest
- **Velocity grid:** 4 stat cards. Pick 4 genuinely notable numbers from this week's research (param counts, pricing milestones, speed, adoption — whatever's most newsworthy).
- **HR & ServiceNow Intelligence:** 4–5 story cards. Each = badge(s) + bold title + **3–5 sentence body with actual analysis** — include the "so what for a Deloitte HRSD consultant" angle — + source link.
- **General AI Hot News:** 5–6 story cards. Same format.
- **Team Submissions:** list Step 1 issues, or empty-state div.
- **POC Ideas highlight box:** 3 specific, actionable POC ideas from this week's research. Each = title + 2–3 sentence description (what to build, why it's timely) + platform + difficulty.
- **Footer:** "Powered by Agentic AI — Claude on Cowork" + pulsing green dot.

### Tab 2 — POC Lab
Copy ALL existing POC cards from the fetched index.html **verbatim**. Never remove any. Only add a new card if this week's research genuinely warrants it. Only update a status badge (Idea → Building → Done) with a specific reason.

### Tab 3 — Archive
- Add a new entry for the issue just archived: previous week's issue number, date, and 3 headline bullets from its content.
- Keep only the 4 most recent entries. Drop the oldest.
- Each entry links to `archive/YYYY-MM-DD.html` using that issue's publication date.

### Tab 4 — Submit Idea
Copy **verbatim** from the fetched index.html. Three cards (Submit POC / Request Investigation / Share Feedback), each linking to a GitHub Issues new-issue URL with pre-filled label + template. Never change this tab.

### Design tokens (preserve exactly — carry the full `<style>` block forward verbatim each week; never hand-rewrite it)
```
--bg: #07080c
--surface: #0d0e15
--surface2: #12131c
--surface3: #191b25
--surface4: #222431
--border: rgba(255,255,255,0.07)
--border-strong: rgba(255,255,255,0.14)
--text: #f3f4f9
--text-dim: #c6c8d6
--muted: #8a8d9f
--muted-dim: #61647a
--green: #00e5a0
--green-bright: #4dffc4
--blue: #5aa9ff
--purple: #b98cff
--orange: #ff9a52
--red: #ff6b6b
--yellow: #ffd166
--pink: #ff7da6
--accent: linear-gradient(90deg, #00e5a0 0%, #2fd9c4 45%, #5aa9ff 100%)
```

**Signature design elements** (they live in the `<style>` block — carrying it verbatim preserves them):
- Fixed gradient accent hairline across the very top of the page (`body::before`).
- Editorial masthead: `h1` at `clamp(34px, 6vw, 58px)`, tight tracking, with a dateline rule beneath it.
- Cards: top-sheen gradient + soft shadow, 3px hover lift. The **first story card in each section is a "featured lead"** (green wash, larger title).
- Glassy sticky tab bar with a gradient underline on the active tab.
- POC hero with gradient-filled numbered badges; pulsing-dot footer.

## Step 5 — Build the archive file

Take the **full HTML of the previous index.html** (fetched in Step 3) and save as `archive/YYYY-MM-DD.html` using the previous issue's publication date (today − 7).

Modify:
- Add a yellow "Archive" ribbon badge to the header.
- Add a "Back to Latest Issue" bar below the header (links to `../index.html`).
- **Strip the tab nav** — archive is a flat single-page view showing just the Latest tab content from that week.

## Step 6 — Write + commit + push

```bash
cd /Users/dmcr/Projects/ai-hr-weekly
# create archive/ if missing (it exists today, but guard anyway)
mkdir -p archive
# write index.html and archive/YYYY-MM-DD.html via the Write tool
git add index.html archive/YYYY-MM-DD.html
git commit -m "Weekly digest — DD Month YYYY"
git push origin main
```

Only run Step 1's issue comment+close loop after `git push` exits 0.

## Step 7 — Draft the team email (Gmail MCP)

Use `gmail_create_draft`:

- **Subject:** `🔥 AI & HR Tech Weekly — [DATE] | HR Technology & Transformation`
- **To:** leave blank (Daniel fills in)
- **Body (HTML):**
  - TL;DR paragraph (3–4 sentences on the week's most important developments)
  - Velocity stats grid (same 4 numbers as the digest header)
  - Top 5 story cards (badge + title + 2-sentence summary + source link)
  - POC Idea of the Week highlight box (the single most actionable POC)
  - Footer links: live digest, Submit Idea, feedback form

## Hard rules

1. **Never remove a POC card** from POC Lab unless Daniel explicitly requests it in the conversation.
2. **Never change the Submit Idea tab** — copy it exactly from the previous index.html every week.
3. **Archive entries: always exactly 4.** Drop the oldest when adding a new one.
4. **Australian English, builder tone, no fluff.** Each story needs a "so what for a Deloitte HRSD practitioner" angle — not just a news summary.
5. **Source links must be real** from actual search results. Never fabricate URLs.
6. **Issue number increments by 1 each week.** Read the current number from the fetched HTML — never hardcode.
7. **Only comment/close GitHub issues after the push succeeds.** If push fails, leave issues open and surface the error.
