# Saved Items Deep Analysis → Action Queue

## Purpose

Convert a batch of Sam's saved items across **X bookmarks, GitHub starred repos, and Readwise (Reader saved articles + highlights)** into a ranked queue of **specific next actions** across his active workstreams. The output is a working action list, not a digest. If a saved item doesn't produce an action or a deliberate "park for later" verdict with a reason, you haven't analyzed it.

All three sources are accepted in any combination. A run can be single-source, any pair, or all three. Three-source batches are the strongest signal because cross-source patterns (a tool tweeted about + a starred repo + a highlighted article on the same topic) carry far more conviction than any single source alone.

## Input

One or more of the following will be supplied:

**X bookmarks:**
- A list of bookmark URLs or tweet IDs
- A JSON dump from the `x-bookmarks` skill or `bird` CLI
- A time range (e.g. "last 30 days of unprocessed bookmarks")

**GitHub stars:**
- A list of repo URLs or `owner/repo` strings
- A JSON dump from `gh api users/$USER/starred --paginate`
- A time range (e.g. "stars from last 60 days")

**Readwise (Reader + highlights):**
- A list of Reader document URLs or document IDs
- A list of highlight IDs
- A JSON dump from `readwise reader-list-documents` or `readwise readwise-list-highlights`
- A time range (e.g. "Reader saves from last 14 days", "highlights from last 30 days")
- A tag filter (e.g. all Reader docs tagged `margin`)

If none is supplied, ask which batch(es) to pull. Don't guess.

## Hard rules (do not violate)

- **`bird` CLI for all X content.** Never `WebFetch` on `x.com` — it returns a 402 login wall. Use `bird read <url>` for single tweets, `bird thread <url>` for threads.
- **`gh` CLI for all GitHub content.** Use `gh repo view <repo> --json description,homepageUrl,stargazerCount,pushedAt,primaryLanguage,repositoryTopics` for metadata and `gh api repos/<owner>/<repo>/readme --jq '.content' | base64 -d` (or `gh repo view <repo> --json readme`) for README content. Use `gh api users/<owner>` for maintainer profile data.
- **`readwise` CLI for all Readwise content.** Use `readwise reader-get-document-details --id <doc_id> --include-html-content` for article body, `readwise reader-get-document-highlights --document-id <doc_id>` for a document's highlights, and `readwise readwise-list-highlights --updated-after <ISO date>` for recent standalone highlights. Honor the `readwise-full-content-strict` rule: for category=podcast/video/pdf/epub OR word_count > 3000 OR summary < 150 chars, **always** extract `html_content` — never compile from the auto-summary alone.
- **Expand before analyzing.** For tweets: read the full thread, quoted tweets, and any linked articles. For repos: read the README, the latest commit date, topics, and primary language. For Reader docs: extract full content per the rule above; for a doc with existing highlights, those highlights ARE the expansion signal — Sam already engaged. For standalone highlights, fetch the parent book/article context if not in the highlight payload. A bookmark of a thread without the thread, a starred repo without the README, or a Reader save without the content (when the rule triggers) is a stub. For linked articles, use `defuddle` or Readwise full-content extraction. Skip pages that 402/paywall — note them, don't fabricate.
- **Default to reviewed, not rejected.** Reject only after reading the underlying content and finding no value. Surface signals (author, repo language, format, topic) are not grounds for dismissal — that's the beacon-inbox-triage rule.
- **Warm paths only for any outreach action.** Cold DMs to strangers — including repo maintainers — are not an action; finding a warm intro path *to* that person is. Sam's strategy v2 is 0% cold channel.
- **Sam writes the final version of any prose.** Drafts, frameworks, and bullet structures are fair game. Do not output finished outreach copy as if it's ready to send.
- **No fabrication.** Don't invent maintainer attributes, company affiliations, repo activity, or any claim that isn't grounded in the source. Acknowledge gaps explicitly with `[unverified]`.
- **No verbatim re-summarizing.** Sam already saw the bookmark or starred the repo. The deliverable is the consequence — what he does because of it.

## Sam's context (use this to score relevance)

Active workstreams, in rough priority order. An action that lights up two of these at once is more valuable than one that hits only one.

1. **Job search** — player-coach product role, AI-native, well-funded, hybrid Northeast. Warm paths only. Conversations > applications. CMF v2.1 spec is the filter; industry is secondary. See `~/cos/strategy/unified-strategy-v2.md`.
2. **Margin** — Tauri + React annotation app. Anything about reading tools, annotation UX, knowledge graphs, local-first apps, Obsidian/Readwise ecosystem, or LLM-augmented reading is on-axis.
3. **Case studies → samzoloth.com** — side-project/writing/consulting work feeds case studies that feed job apps. Look for saved content that supports an existing case study or seeds a new one.
4. **Writing & publishing** — concepts, frameworks, or contrarian takes worth turning into a post that grows surface area.
5. **Operating system** — tools, scripts, skills, agent patterns that compound across sessions. Starred repos that improve Sam's CLI/agent toolchain count strongly here.

## Process

Work in four passes. Don't shortcut.

### Pass 1 — Inventory

For each saved item, capture:

**Common fields (all sources):**
- Source URL
- Author / owner
- Date (created, starred, saved, or highlighted)
- Format (single tweet / thread / quote-tweet / linked article / GitHub repo / Reader article / Reader podcast / Reader video / Reader PDF / Reader epub / standalone highlight)
- One-line honest restatement of the core claim, artifact, or repo purpose

**X-bookmark-specific:**
- The bookmark itself (what Sam saved)
- The expanded content (full thread, linked article body, quoted post)

**Repo-specific:**
- Primary language and topics
- Star count, last push date, archived status
- README summary (what it does, who it's for, how to install/run)
- Maintainer profile signal if relevant (sole maintainer, org-backed, recent activity)

**Readwise-specific:**
- Category (article / book / podcast / video / pdf / epub / tweet / email)
- Reader location (new / later / archive / shortlist / feed)
- Whether Sam has highlighted within it (and how many highlights — engagement signal)
- Tags Sam applied in Reader
- `first_opened_at` (null = unseen, non-null = seen)
- For standalone highlights: the highlight text, any note Sam attached, the parent book/article title and author
- Expanded content per `readwise-full-content-strict`

Skip nothing. If `bird`, `gh`, or `readwise` fails on an item, note it and continue.

### Pass 2 — Classify

For each saved item, assign **one primary category and zero-or-more secondary categories**:

- **Person** — a specific human worth a warm-path conversation (tweet author, repo maintainer, contributor)
- **Company / role signal** — a hiring company, team, or pattern that matches the CMF v2.1 structural spec (a starred repo's parent org counts)
- **Tool / method** — something Sam could *use* this week (skill, CLI, framework, prompt pattern, library). **Most starred repos land here.** A repo as Tool requires a concrete "Sam can do X with this in ≤N minutes" hook — clone-and-run, fork-and-adapt, or steal-the-pattern.
- **Margin input** — a feature idea, UX pattern, competitor signal, or user pain relevant to Margin. Repos that are direct Margin-adjacent (annotation libs, reading tools, knowledge graphs, local-first sync) belong here, not Tool.
- **Case study seed** — content that supports or originates a samzoloth.com case study
- **Writing seed** — a contrarian take, framework, or angle worth a post under Sam's voice
- **Reading queue** — long-form worth reading in full but not yet actionable. Includes deep-dive READMEs/docs that aren't immediately usable.
- **Concept** — a framework or mental model worth internalizing; no immediate task
- **Cold storage** — verified-as-reviewed but no action. Must include a reason.

Repo-specific classification rules:
- **Archived or last-push >18 months ago:** default to Reading or Cold storage unless there's a clear "steal the pattern" or "fork and revive" justification.
- **<50 stars and no clear differentiator:** default to Reading or Cold storage.
- **Fits Margin's domain:** route to Margin input, not Tool — even if it's also usable as a tool.

Readwise-specific classification rules:
- **Reader doc with highlights:** the doc has already passed Sam's first filter. Default to Concept, Writing seed, or Case study — not Reading queue (he already read it). Use the highlights themselves to determine which.
- **Reader doc, unread (first_opened_at is null), word_count > 3000:** default to Reading queue unless the content explicitly suggests Tool/Margin/Person.
- **Standalone highlight:** primary category is almost always Concept or Writing seed. Treat the highlight as a compressed thesis statement Sam has already endorsed.
- **Reader category=podcast/video/pdf/epub:** never classify from summary alone; the full transcript or extracted content is the source of truth.
- **Reader doc in shortlist:** Sam manually curated this. Apply higher relevance weight in Pass 4 scoring than feed/later docs.

### Pass 3 — Cross-cluster

Look across the batch for patterns. Cross-source patterns are the highest-signal findings; a single item is weaker evidence than three independent sources converging. **Three-source patterns (X + GitHub + Readwise) are the strongest possible signal — they represent the same thesis arriving through three independent curation channels.**

Report any of the following:
- A tool mentioned in a bookmark, represented in a starred repo, AND written about in a saved/highlighted article (3-way convergence — flag as headline)
- A tool tweeted about and starred (2-way X+GH convergence)
- A concept Sam tweeted-saved and also highlighted in a long-form article (2-way X+Readwise convergence)
- A repo Sam starred whose maintainer or topic is also covered in a saved article (2-way GH+Readwise convergence)
- A person who appears as both a bookmark author and a repo maintainer, or as both a tweet author and a highlighted article author (strong Person candidate)
- Two or more saved items pointing at the same company, person, or tool across sources
- A recurring concept showing up across tweets, READMEs, and article highlights from different communities
- A contradiction between two saved items
- A latent thesis Sam is building toward without realizing it (a cluster of saves pointing at the same problem space across sources)

### Pass 4 — Score and write the action queue

For each item that survives to "action," produce a record with this shape:

```
ACTION: <imperative verb phrase, ≤12 words>
Category: <Person | Tool | Margin | Case study | Company | Writing | Reading | Concept>
Source: <bookmark URL or repo URL>
Why it matters to Sam: <1-2 sentences tying to a specific workstream. Be concrete, not "interesting for product work.">
Next step: <the literal next thing to do, with the tool/path. e.g. "Clone repo, run quickstart, evaluate whether it slots into Margin's PDF pipeline. Notes to ~/Projects/margin/notes/<repo-name>.md." or "Draft 80-word outreach to <name> referencing <specific shared context>. Save to ~/cos/outreach/<name>.md.">
Effort: <S = under 15 min | M = under 1 hr | L = a session>
Confidence: <High | Medium | Low — based on source quality, maintenance signal, and Sam-fit evidence>
ETA: <a real date within the next 14 days, or "park: <date>" with a re-surface date>
```

Then rank the full list using this priority:

1. **Warm-path conversations** (Person category with a plausible intro path) — these are scarce and time-sensitive.
2. **3-source convergence items** (any cluster that appeared across X + GitHub + Readwise in Pass 3) — these are the highest-conviction signals in the batch.
3. **2-source convergence items** (mixed X+GH, X+Readwise, or GH+Readwise clusters).
4. **Margin or case-study items that unblock work in flight.**
5. **Highlighted-article-derived items** (Concepts or Writing seeds anchored to a specific Readwise highlight Sam already endorsed) — higher confidence than freshly-saved-but-unread items.
6. **Tools / repos** Sam can apply this week with S effort.
7. **Writing seeds** that map to an existing case study or a published-this-quarter goal.
8. **Reading queue and concepts** at the bottom.

Cap the queue at **15 actions per batch.** If more survive, demote the weaker ones to a "next batch" section with their park dates. Volume kills follow-through.

## Output format

Produce one Markdown document with these sections, in this order:

```
# Saved Items Analysis — <date range>

## Summary
- X bookmarks processed: <N>
- Starred repos processed: <N>
- Reader docs processed: <N>
- Standalone highlights processed: <N>
- Actions surfaced: <N>
- Parked: <N>
- Cold storage: <N>
- Failed to expand: <N>

## Cross-cluster patterns
<bulleted list from Pass 3. Lead with any 3-source convergences (X + GitHub + Readwise), then 2-source, then within-source patterns. Empty list is acceptable if nothing converges — say so, don't pad.>

## Action queue
<numbered list of action records, ranked per Pass 4.>

## Parked
<saved items worth keeping in mind but not acting on now, with re-surface date and reason. One line each.>

## Cold storage
<saved items reviewed and dropped, with a one-line reason. One line each.>

## Failed to expand
<saved items where the underlying content could not be retrieved, with the error. One line each.>
```

## Common mistakes to avoid

- **Summarizing the saved item.** Sam saved it; he saw the surface. The deliverable is the *consequence* — what he does because of it.
- **"Interesting for Margin" with no specific action.** Either name the feature, the user pain, or the competitor signal — or move it to Concept.
- **Repo with no maintenance signal treated as a live Tool.** A repo with last commit >12 months ago is Reading or Cold storage unless there's a justified "steal the pattern" angle.
- **Conflating star count with relevance to Sam.** A 30k-star repo Sam can't realistically use this week is Reading. A 200-star repo that slots directly into Margin is Tool.
- **Generating polished outreach copy** — including to repo maintainers. Provide angle, shared context, and draft skeleton. Sam writes the final version.
- **Loading every saved item into the action queue.** A ranked list of 15 strong actions beats a flat list of 60.
- **Treating "this person is interesting" as a Person action.** A Person action requires: a plausible warm-path intro, a specific reason to talk now, and the angle of conversation. Without those, it's Cold storage or Reading.
- **Skipping the expansion step on threads, articles, or READMEs.** A bookmark of a thread is a bookmark of the *whole thread*; a starred repo without the README is a stub.
- **Fabricating maintainer or company context.** If you can't verify a claim ("they raised a Series A in March," "they work at Anthropic"), drop the claim or mark it `[unverified]`.
- **Cold-storage dumps without reasons.** A saved item Sam saved had some signal; explain what made you drop it so future passes don't re-litigate.
- **Padding the cross-cluster section.** If nothing converged, say so. Forced patterns are worse than no patterns.
- **Treating a starred repo as if Sam will deploy it.** Default action verbs for repos are "Try" (S), "Steal the pattern from" (S), "Fork and adapt for X" (M), or "Build a wrapper integrating with Margin" (L). Not "Use this in production."
- **Treating an unread Reader save as if Sam read it.** A doc with `first_opened_at: null` is intent-to-read, not read. Don't infer Sam endorses its claims; classify as Reading queue or Concept-pending unless the content itself produces a Tool/Margin hook.
- **Compiling from a Readwise auto-summary.** The `readwise-full-content-strict` rule is non-negotiable for podcast/video/pdf/epub OR word_count > 3000 OR summary < 150 chars. Extract `html_content` or treat the item as failed-to-expand.
- **Treating a highlight as the same signal as a save.** A highlight is a *stronger* signal — Sam already read the parent, picked the passage, and chose to preserve it. Highlights should usually become Concepts or Writing seeds with high confidence, not Reading queue items.

## Tools and paths to use

- `bird read <url>` — single tweet content
- `bird thread <url>` — full thread
- `gh repo view <repo> --json description,homepageUrl,stargazerCount,pushedAt,primaryLanguage,repositoryTopics,isArchived` — repo metadata
- `gh api repos/<owner>/<repo>/readme --jq '.content' | base64 -d` — README content
- `gh api users/<owner>` — maintainer profile data
- `gh api users/$USER/starred --paginate` — Sam's full stars list
- `readwise reader-list-documents --location new --updated-after <ISO>` — recent Reader saves
- `readwise reader-get-document-details --id <doc_id> --include-html-content` — full article body (required when the strict rule triggers)
- `readwise reader-get-document-highlights --document-id <doc_id>` — highlights on a saved doc
- `readwise readwise-list-highlights --updated-after <ISO>` — recent standalone highlights across the corpus
- `readwise readwise-search-highlights --query "<term>"` — search the highlight corpus for cross-cluster pattern verification
- `x-bookmarks` skill — pull a batch of saved bookmarks
- `defuddle` skill — clean extraction of linked article content
- `beacon` skill — bookmark + star intake; if a Person or Company action exists, beacon may already have an item for it; check before duplicating
- `siftly-matcher` skill — matches X bookmarks and starred repos against Sam's local systems and projects; useful for cross-source pattern detection
- `~/cos/strategy/unified-strategy-v2.md` — read once at the start of any session that includes Person or Company actions, to pick up the current CMF spec and channel rules
- Linear `linear-manager` skill — if a Company action passes the CMF filter, surface whether a pipeline issue already exists

## What to ask before starting

Only ask if these are genuinely unresolvable from input:

- Which batch(es) of saved items? (X bookmarks, GitHub stars, Readwise Reader docs, Readwise highlights — any combination, with URLs, time ranges, tag filters, or skill outputs)
- Any workstream to weight more heavily this run? (default: balanced across the five)
- Output location: stdout, vault file at `resources/saved-items/<date>.md`, or both?

Otherwise, start.
