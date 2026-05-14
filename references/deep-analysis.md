# Deep Analysis of X Bookmarks → Action Queue

## Purpose

Convert a batch of Sam's saved X (Twitter) bookmarks into a ranked queue of **specific next actions** across his active workstreams. The output is a working action list, not a digest. If a bookmark doesn't produce an action or a deliberate "park for later" verdict with a reason, you haven't analyzed it.

## Input

One of the following will be supplied:
- A list of bookmark URLs or tweet IDs
- A JSON dump from the `x-bookmarks` skill or `bird` CLI
- A time range (e.g. "last 30 days of unprocessed bookmarks")

If none is supplied, ask which batch to pull. Don't guess.

## Hard rules (do not violate)

- **`bird` CLI for all X content.** Never `WebFetch` on `x.com` — it returns a 402 login wall. Use `bird read <url>` for single tweets, `bird thread <url>` for threads.
- **Expand before analyzing.** Read the full thread, quoted tweets, and any linked articles. A bookmark of a thread without reading the thread is a stub, not analysis. For linked articles, use `defuddle` or Readwise full-content extraction. Skip pages that 402/paywall — note them, don't fabricate.
- **Default to reviewed, not rejected.** Reject a bookmark only after reading the underlying content and finding no value. Surface signals (author, format, topic) are not grounds for dismissal — that's the beacon-inbox-triage rule.
- **Warm paths only for any outreach action.** Cold DMs to strangers are not an action; finding a warm intro path *to* that person is. Sam's strategy v2 is 0% cold channel.
- **Sam writes the final version of any prose.** Drafts, frameworks, and bullet structures are fair game. Do not output finished outreach copy as if it's ready to send.
- **No fabrication.** Don't invent attributes about people, companies, or claims that aren't in the source. Acknowledge gaps explicitly.
- **No verbatim re-summarizing.** Sam already read the bookmark; the surface text is not the deliverable. Pull the consequence.

## Sam's context (use this to score relevance)

Active workstreams, in rough priority order. An action that lights up two of these at once is more valuable than one that hits only one.

1. **Job search** — player-coach product role, AI-native, well-funded, hybrid Northeast. Warm paths only. Conversations > applications. CMF v2.1 spec is the filter; industry is secondary. See `~/cos/strategy/unified-strategy-v2.md`.
2. **Margin** — Tauri + React annotation app. Anything about reading tools, annotation UX, knowledge graphs, local-first apps, Obsidian/Readwise ecosystem, or LLM-augmented reading is on-axis.
3. **Case studies → samzoloth.com** — side-project/writing/consulting work feeds case studies that feed job apps. Look for bookmark content that supports an existing case study or seeds a new one.
4. **Writing & publishing** — concepts, frameworks, or contrarian takes worth turning into a post that grows surface area.
5. **Operating system** — tools, scripts, skills, agent patterns that compound across sessions. Margin sessions count here; so do CLI/skill improvements.

## Process

Work in four passes. Don't shortcut.

### Pass 1 — Inventory

For each bookmark, capture:
- URL, author, date, format (single tweet / thread / quote-tweet / linked article)
- The bookmark itself (what Sam saved)
- The expanded content (full thread, linked article body, quoted post)
- An honest one-line restatement of the core claim or artifact

Skip nothing. If `bird` fails on an item, note it and continue.

### Pass 2 — Classify

For each bookmark, assign **one primary category and zero-or-more secondary categories**:

- **Person** — a specific human worth a warm-path conversation
- **Company / role signal** — a hiring company, team, or pattern that matches the CMF v2.1 structural spec
- **Tool / method** — something Sam could *use* this week (skill, CLI, framework, prompt pattern)
- **Margin input** — a feature idea, UX pattern, competitor signal, or user pain relevant to Margin
- **Case study seed** — content that supports or originates a samzoloth.com case study
- **Writing seed** — a contrarian take, framework, or angle worth a post under Sam's voice
- **Reading queue** — long-form worth reading in full but not yet actionable
- **Concept** — a framework or mental model worth internalizing; no immediate task
- **Cold storage** — verified-as-reviewed but no action. Must include a reason.

### Pass 3 — Cross-cluster

Look across the batch for patterns. Cross-source patterns are the highest-signal findings; a single tweet is weaker evidence than three independent sources converging.

Report any of the following:
- Two or more bookmarks pointing at the same company, person, or tool
- A recurring concept showing up in different communities
- A contradiction between two bookmarks Sam saved
- A latent thesis Sam is building toward without realizing it

### Pass 4 — Score and write the action queue

For each item that survives to "action," produce a record with this shape:

```
ACTION: <imperative verb phrase, ≤12 words>
Category: <Person | Tool | Margin | Case study | Company | Writing | Reading | Concept>
Source: <bookmark URL>
Why it matters to Sam: <1-2 sentences tying to a specific workstream. Be concrete, not "interesting for product work.">
Next step: <the literal next thing to do, with the tool/path. e.g. "Draft 80-word outreach to <name> referencing <specific shared context>. Save to ~/cos/outreach/<name>.md.">
Effort: <S = under 15 min | M = under 1 hr | L = a session>
Confidence: <High | Medium | Low — based on source quality and Sam-fit evidence>
ETA: <a real date within the next 14 days, or "park: <date>" with a re-surface date>
```

Then rank the full list using this priority:

1. **Warm-path conversations** (Person category with a plausible intro path) — these are scarce and time-sensitive.
2. **Pattern-cluster items** (anything that showed up 2+ times in Pass 3).
3. **Margin or case-study items that unblock work in flight.**
4. **Tools / methods** Sam can apply this week with S effort.
5. **Writing seeds** that map to an existing case study or a published-this-quarter goal.
6. **Reading queue and concepts** at the bottom.

Cap the queue at **15 actions per batch.** If more survive, demote the weaker ones to a "next batch" section with their park dates. Volume kills follow-through.

## Output format

Produce one Markdown document with these sections, in this order:

```
# X Bookmark Analysis — <date range>

## Summary
- Bookmarks processed: <N>
- Actions surfaced: <N>
- Parked: <N>
- Failed to expand: <N>

## Cross-cluster patterns
<bulleted list from Pass 3. Empty list is acceptable if nothing converges — say so, don't pad.>

## Action queue
<numbered list of action records, ranked per Pass 4.>

## Parked
<bookmarks worth keeping in mind but not acting on now, with re-surface date and reason. One line each.>

## Cold storage
<bookmarks reviewed and dropped, with a one-line reason. One line each.>

## Failed to expand
<bookmarks where the underlying content could not be retrieved, with the error. One line each.>
```

## Common mistakes to avoid

- **Summarizing the bookmark.** Sam saved it; he saw the surface. The deliverable is the *consequence* — what he does because of it.
- **"Interesting for Margin" with no specific action.** Either name the feature, the user pain, or the competitor signal — or move it to Concept.
- **Generating polished outreach copy.** Provide the angle, the shared context, and a draft skeleton. Sam writes the final version.
- **Loading every bookmark into the action queue.** A ranked list of 15 strong actions beats a flat list of 60.
- **Treating "this person is interesting" as a Person action.** A Person action requires: a plausible warm-path intro, a specific reason to talk now, and the angle of conversation. Without those, it's Cold storage or Reading.
- **Skipping the expansion step on threads or linked articles.** A bookmark of a thread is a bookmark of the *whole thread*; analyze accordingly.
- **Fabricating author or company context.** If you can't verify a claim about a person ("they raised a Series A in March"), drop the claim or mark it `[unverified]`.
- **Cold-storage dumps without reasons.** A bookmark Sam saved had some signal; explain what made you drop it so future passes don't re-litigate.
- **Padding the cross-cluster section.** If nothing converged, say so. Forced patterns are worse than no patterns.

## Tools and paths to use

- `bird read <url>` — single tweet content
- `bird thread <url>` — full thread
- `x-bookmarks` skill — pull a batch of saved bookmarks
- `defuddle` skill — clean extraction of linked article content
- `beacon` skill — if a Person or Company action exists, beacon may already have an item for it; check before duplicating
- `~/cos/strategy/unified-strategy-v2.md` — read once at the start of any session that includes Person or Company actions, to pick up the current CMF spec and channel rules
- Linear `linear-manager` skill — if a Company action passes the CMF filter, surface whether a pipeline issue already exists

## What to ask before starting

Only ask if these are genuinely unresolvable from input:

- Which batch of bookmarks? (URLs, time range, or x-bookmarks skill output)
- Any workstream to weight more heavily this run? (default: balanced across the five)
- Output location: stdout, vault file at `resources/x-bookmarks/<date>.md`, or both?

Otherwise, start.
