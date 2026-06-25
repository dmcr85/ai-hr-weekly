---
name: ai-hr-weekly
description: Weekly AI & HR Tech digest pipeline — research, build index.html + archive, commit/push to GitHub, draft team email. Runs end-to-end in a single Cowork session on the Mac.
---

# Weekly AI & HR Tech Digest — Pipeline Spec (Cowork / Mac-native edition)

This is the canonical spec. Run it end-to-end in ONE session on schedule. Every step, tool call, and file op is defined here — ask no questions, just execute.

**Repo (on the Mac):** `/Users/dmcr/Projects/ai-hr-weekly` — remote `dmcr85/ai-hr-weekly`
**Live index (source of truth for the current issue):** https://raw.githubusercontent.com/dmcr85/ai-hr-weekly/main/index.html
**Published site:** https://dmcr85.github.io/ai-hr-weekly/
**GitHub issues:** https://github.com/dmcr85/ai-hr-weekly/issues

## Runtime model — READ THIS FIRST

This job runs on Cowork under Daniel's Max subscription (no metered API cost). There are TWO different shells available, and using the wrong one is what makes pushes fail:

- **The default workspace bash is a sandboxed Linux VM.** It has NO GitHub credentials and NO `gh`. NEVER run `git` or `gh` there — `git push` fails with "could not read Username". Use it only for web search and scratch work.
- **The Mac itself has the working toolchain.** Run ALL `git` and `gh` commands ON THE MAC via the `Control_your_Mac` osascript tool, e.g. `do shell script "export PATH=$PATH:/opt/homebrew/bin:/usr/local/bin; cd /Users/dmcr/Projects/ai-hr-weekly && <command> 2>&1"`. The Mac has `gh` (logged in as `dmcr85`), git push via the `osxkeychain` helper, and the commit identity `Daniel Correal <danielcorreal85@gmail.com>` already configured. This is how the pipeline has always published.
- **File writes** to `index.html` and the archive file: use the Write tool against `/Users/dmcr/Projects/ai-hr-weekly/...` (it writes the real repo). Then stage/commit/push on the Mac via osascript.

When running a Mac command, always capture `2>&1` and check the result. On any failure that prevents publishing, go to **Step 9** (alert) and stop — do not press on.

## Dates

- **Working date** = today (publication date). Format: `DD Month YYYY` in copy, `YYYY-MM-DD` in filenames. Get today from the Mac (`date +%F`), don't assume.
- **Previous issue date** = today − 7 days. Used for the archive filename.

## Step 0 — Preflight + sync (on the Mac)

Via osascript on the Mac, in the repo:
```
export PATH=$PATH:/opt/homebrew/bin:/usr/local/bin
cd /Users/dmcr/Projects/ai-hr-weekly
rm -f .git/index.lock                 # clear any stale lock
git fetch origin 2>&1
git pull --ff-only origin main 2>&1   # MUST sync before building — a behind clone fails to push
gh auth status 2>&1 | head -3
```
If `git pull` cannot fast-forward, or `gh auth status` is not logged in, jump to **Step 9** with the reason and stop. Pulling first is mandatory: the most common cause of a failed run is pushing from a clone that is behind `origin/main`.

## Step 1 — Pull team submissions from GitHub (gh on the Mac)

```
gh issue list --repo dmcr85/ai-hr-weekly --label poc-idea,investigation-request,feedback --state open --json number,title,body,labels
```
Capture each issue's `number`, `title`, `body`, `labels` for the **Team Submissions** section. If none, render the empty-state div. Do NOT comment/close yet — that is Step 8, only after a verified publish.

## Step 2 — Research (11 web searches, last 7 days)

Run all 11 (web search in the sandbox is fine). Substitute `[month year]` with the current month + year.

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

1. **Current issue number** → increment by 1 for the new issue. Never hardcode.
2. **Full POC Lab tab content** → carry forward verbatim.
3. **Archive tab entries** → keep the 4 most recent; the oldest will be dropped.
4. **Submit Idea tab** → copy verbatim. Never modify.
5. **The entire `<style>` block** AND the Google Fonts `<link>` tags in `<head>` → carry forward verbatim (see Step 4).

Keep the full fetched HTML — Step 5 turns it into the archive page.

## Step 4 — Build the new index.html

Single self-contained HTML file: all CSS/JS inline. The ONE allowed external resource is the Google Fonts `<link>` in `<head>` (Space Grotesk + IBM Plex Sans + IBM Plex Mono) — carry it forward verbatim, never drop it. Write the file to `/Users/dmcr/Projects/ai-hr-weekly/index.html` with the Write tool.

### Masthead (modern "wire" header — NOT a badge row, NOT centred)
Left-aligned, terminal-style:
- `.wire` mono bar — `Issue <N>` (`.wire-issue`, accent) `//` `<date>` `//` `Deloitte HRSD Practice` `//` `Signal AI × HR` (`.wire-live`, trailing accent dot). Separators are mono `//`.
- `.nameplate` `<h1>` "AI &amp; HR Tech Weekly" — large LEFT-aligned grotesk (Space Grotesk), tight negative tracking.
- `.standfirst` — one sans positioning line, left-aligned, above a hairline.
No "Auto-compiled by Claude" badge, no issue pill, NO emoji. (Transparency note lives in the footer colophon.)

### Section nav (sticky top, 4 tabs; mono uppercase; active tab in accent emerald with a 2px underline)
Latest · POC Lab · Archive · Submit Idea — mono, letter-spaced, NO emoji. Keep the `.kbd-hint` figure on each tab.

### Tab 1 — Latest
- **By the Numbers:** a `.kicker-numbers` mono label above the 4-column `.velocity-grid` — 4 genuinely notable numbers (param counts, pricing milestones, speed, adoption) as white cards with a small accent top-tab. Figures render in **ink grotesk** (`--display`); never add inline `style="color:…"` to `.velocity-value`.
- **HR & ServiceNow Intelligence:** 4–5 story cards. Each = badge(s) + bold title + **3–5 sentence body with actual analysis**, including the "so what for a Deloitte HRSD consultant" angle, + source link.
- **General AI Hot News:** 5–6 story cards. Same format.
- **Team Submissions:** list Step 1 issues, or empty-state div.
- **POC Ideas highlight box:** 3 specific, actionable POC ideas from this week's research. Each = title + 2–3 sentence description (what to build, why timely) + platform + difficulty. The flagship of the issue — deepest analysis: ServiceNow HRSD capability + the week's AI developments + the concrete business opportunity.
- **Footer colophon:** `.footer-wordmark` (grotesk) + `.footer-tagline` + a mono `.footer-colophon` line — `Issue <N> · <date> · Compiled with Claude on Cowork`. No pulsing dot, no "Powered by Agentic AI" badge, no emoji.

### Tab 2 — POC Lab
Copy ALL existing POC cards from the fetched index.html **verbatim**. Never remove any. Add a new card only if this week's research genuinely warrants it. Update a status badge (Idea → Building → Done) only with a specific reason.

### Tab 3 — Archive
- Add an entry for the issue just archived: previous week's issue number, date, 3 headline bullets.
- Keep only the 4 most recent entries. Drop the oldest.
- Each entry links to `archive/YYYY-MM-DD.html` using that issue's publication date.

### Tab 4 — Submit Idea
Copy **verbatim** from the fetched index.html. Three cards (Submit POC / Request Investigation / Share Feedback), each linking to a GitHub Issues new-issue URL with pre-filled label + template. Never change this tab.

### Design tokens (modern "Wire" — carry the full `<style>` block forward verbatim; never hand-rewrite it)
Cool near-white, white cards, ONE bold accent (deep emerald), mono "wire" details, grotesk display. Core tokens:
```
--paper:  #f5f5f3   (cool near-white canvas)   --card:  #ffffff  (white cards)
--ink:    #15161a   (cool near-black)          --ink-2: #33353c  (body text)
--steel:  #565a63   (secondary)  --mist: #8a8f99  (metadata)  --faint: #b6bac1
--line:   #e4e4e7   --line-2: #d2d4d9          (cool hairlines)
--accent: #0a7d54   (deep emerald — the ONLY bold note; #07623f on hover)
--display: 'Space Grotesk'              (wordmark, headlines, numerals)
--sans:    'IBM Plex Sans' / system-ui  (body)
--mono:    'IBM Plex Mono'              (wire: datelines, indices, kickers, source credits, labels)
--r: 12px / --r-sm: 8px                 (feature radius — deliberately NOT zero)
```
The accent is deliberately **deep** emerald (not neon mint) so small mono text stays legible on white. No neon, no gradients, no glow, no glassmorphism, no dark mode, no warm cream, no broadsheet serif — these are the three AI-default looks this design avoids.

### Fonts — in `<head>` (carry verbatim, never drop)
```
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Space+Grotesk:wght@400;500;600;700&family=IBM+Plex+Sans:wght@400;500;600&family=IBM+Plex+Mono:wght@400;500;600&display=swap" rel="stylesheet">
```

**Signature design elements** (they live in the `<style>` block + the font `<link>` — carrying them forward preserves the look):
- A thin solid accent (emerald) band fixed across the very top (`body::before`).
- Modern "wire" masthead: left-aligned grotesk `.nameplate` over a mono `.wire` metadata bar with `//` separators and a live accent dot.
- Mono "wire" system threaded through the page: indexed section eyebrows (`.section-title::before` auto-numbers `01 / 02 / 03` via a CSS counter), mono kickers, `↳` source credits (`.story-link::before`).
- Indicators are **white cards** (`--r` radius) with a small accent top-tab and big grotesk figures.
- On **Latest**, articles sit in a fixed **reading column** (`--col`, ~672px): the headline, body, and the dividing rules all share that width and align — never a full-width headline over a narrow paragraph. The indicators strip and the POC flagship stay full-width. Stories are a **ruled column, not boxes**. The **first story in each section is the lead**: bigger grotesk headline + a vertical **accent bar** on its left (`.story-card:first-child::after`) — NOT a drop cap.
- Badges are mono **kickers** in `[ brackets ]` above each headline; `NEW/HOT/BREAKING` in accent, the rest muted.
- POC flagship: a white card with a solid **accent header bar** (white text) and large grotesk numerals.
- Footer is a mono **colophon**. Subtle radius on feature cards (anti-broadsheet); NO emoji anywhere.

## Step 5 — Build the archive file

Take the **full HTML of the previous index.html** (fetched in Step 3) and write it to `/Users/dmcr/Projects/ai-hr-weekly/archive/YYYY-MM-DD.html` using the previous issue's publication date (today − 7).

Modify:
- Mark it as archived in the masthead — set the `.wire` `.wire-live` text to "Archived Issue" (keep the accent dot). No off-palette ribbon.
- Add a "Back to Latest Issue" bar below the header (links to `../index.html`).
- **Strip the tab nav** — archive is a flat single-page view showing just that week's Latest content.

## Step 6 — Commit + push (on the Mac, via osascript)

```
export PATH=$PATH:/opt/homebrew/bin:/usr/local/bin
cd /Users/dmcr/Projects/ai-hr-weekly
git add index.html archive/YYYY-MM-DD.html
git commit -m "Weekly digest — DD Month YYYY"
git push origin main 2>&1
```
Check the push output. If it is rejected (e.g. non-fast-forward) or otherwise fails, jump to **Step 9** with the error and stop — do NOT close issues, do NOT draft the email as if it shipped. (Step 0 already pulled, so a fast-forward push is expected.)

## Step 7 — Verify the publish is live (liveness check)

Re-fetch the published `index.html` from the raw URL with a cache-buster (append `?t=<epoch>`). Confirm the **new issue number** appears. Retry up to 3 times, ~20s apart. If the new number is NOT live, treat the run as FAILED → **Step 9**. This is the check that turns a silent failure into a loud one.

## Step 8 — Close the team submissions (gh on the Mac, only now)

Only after Step 7 confirms the new issue is live, for each issue included in Step 1:
```
gh issue comment <number> --repo dmcr85/ai-hr-weekly --body "Included in digest for <DD Month YYYY>"
gh issue close   <number> --repo dmcr85/ai-hr-weekly
```
If this step errors it is non-fatal (the digest already shipped) — note it in the summary, don't fail the whole run.

## Step 9 — Failure alert  ⟵ runs on ANY abort above

If any step aborts the run, do BOTH of the following, then STOP (never report success you didn't achieve):

1. Send an iMessage via the iMessage connector to **+61402033675**:
   > ⚠️ AI-HR Weekly run FAILED at <step> on <DD Month YYYY>. Reason: <short reason>. Nothing was published/closed past that point. Open Cowork to re-run.
2. SEND (not draft) the same message as an email via the Mac Mail app (see Step 10 for the send mechanism), subject `⚠️ AI-HR Weekly run FAILED — <DD Month YYYY>`, to **danielcorreal85@gmail.com** and **dcorreal@deloitte.com.au**.

## Step 10 — Send the digest email (Mac Mail app — NOT a draft)

Only on a successful, verified publish. The Gmail connector can only draft, not send, so SEND via the Mac Mail app using osascript. Mail's iCloud account is enabled (sends from danielcorreal85@icloud.com); the Google account is disabled, so do not try to send from gmail.

Send TO: **danielcorreal85@gmail.com** AND **dcorreal@deloitte.com.au** (Daniel, both inboxes — not the team; Daniel forwards to the team himself).

AppleScript pattern (plain-text body — Mail scripting does not do reliable HTML, so write a clean, readable text email; the styled version lives on the site):
```
tell application "Mail"
  set m to make new outgoing message with properties {subject:"🔥 AI & HR Tech Weekly — <DATE>", content:"<body>", visible:false}
  tell m
    make new to recipient at end of to recipients with properties {address:"danielcorreal85@gmail.com"}
    make new to recipient at end of to recipients with properties {address:"dcorreal@deloitte.com.au"}
  end tell
  send m
end tell
```

Body content:
- One-line link first: **Live digest: https://dmcr85.github.io/ai-hr-weekly/**
- TL;DR paragraph (3–4 sentences on the week's most important developments)
- The 4 "By the Numbers" figures (one per line)
- Top 5 stories: badge + title + one-line summary + source URL
- POC Idea of the Week (the single most actionable POC: what to build + why now)
- Footer: Submit Idea / feedback form links

Daniel wants this email every week — do not skip it. (Note: mail sent from iCloud to the Deloitte address may land in Deloitte's junk/quarantine; that's an external filter, not a run failure.)

## Step 11 — Run summary

End with a short summary: new issue number, commit pushed (yes/no + short SHA if available), liveness confirmed (yes/no), issues closed, email draft created (yes/no), and anything non-fatal that went wrong.

## Hard rules

1. **Run git and gh ON THE MAC via osascript** — never in the Linux sandbox (no creds there).
2. **Always `git pull --ff-only` in Step 0 before building** — a behind clone is the classic push failure.
3. **Never remove a POC card** from POC Lab unless Daniel explicitly requests it.
4. **Never change the Submit Idea tab** — copy it exactly from the previous index.html every week.
5. **Archive entries: always exactly 4.** Drop the oldest when adding a new one.
6. **Australian English, builder tone, no fluff.** Each story needs a "so what for a Deloitte HRSD practitioner" angle — analysis, not just a summary.
7. **Source links must be real** from actual search results. Never fabricate URLs.
8. **Issue number increments by 1 each week.** Read it from the live page — never hardcode.
9. **Comment/close issues only after Step 7 confirms the publish is live.** If publish/liveness fails, leave issues open and fire the Step 9 alert.
10. **The 3 weekly POC ideas are the flagship** — deepest analysis of the bunch.
