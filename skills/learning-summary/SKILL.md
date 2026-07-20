---
name: learning-summary
description: Distill the current coding or AI session into deep, teachable notes and grow a personal concept knowledge base in your learning-center repo. Use at the end of a session to capture what was learned; commits locally and asks before pushing.
disable-model-invocation: true
---

Turn a working session into durable, teachable knowledge. The move that makes this skill
worth running: **surface the concept beneath the task** — file "exponential backoff", not
"added a retry loop". Do that every run and the knowledge base compounds.

Three principles shape the output:

- **Resource-first** — find real sources before you explain; never teach a concept purely
  from parametric memory, and cite those sources *inline* in the explanation so every claim
  is traceable.
- **Storage over fluency** — write for long-term recall, not just a nice read: every
  concept file ends in retrieval-practice questions, which feed a weekly review file you
  actually study from.
- **Only what's worth learning** — a session with nothing genuinely transferable produces
  **nothing**. Never manufacture concepts to hit a quota. Keep lessons concise; cut noise.

## The lessons repo

All output goes to **one dedicated repo, never the current project**:

- **LC_DIR** — the local clone of the lessons repo. It is **not** hardcoded to home;
  resolve it at the start of every run (see step 1). It comes from the
  `LEARNING_CENTER_DIR` environment variable, and only falls back to
  `$HOME/learning-center` when that variable is unset.
- **LC_REMOTE** = `https://github.com/TGubbels/learning-center.git`

```
LC_DIR/
  INDEX.md                              running overview + suggestions + review/session links
  concepts/<category>/<slug>.md          one standalone lesson per concept — STABLE path, never moves
  sessions/<week>/YYYY-MM-DD-<topic>.md  per-session recap linking to concepts
  review/<week>.md                       that week's recall questions, for 5–10 min study bursts
```

**Concepts are filed by category on a stable path and never move.** The week(s) a concept
was touched live in its YAML **frontmatter** (`weeks`, `last_seen`, `times`), not in the
folder — so there is never a `git mv`, never an inbound link to repoint. Categories keep
related concepts together (`concurrency`, `databases`, `http`, `testing`, `ai-llm`,
`architecture`, …); a concept with no natural category sits loose directly under `concepts/`.

**Sessions and review files are filed by week** (`<week>` = ISO week, e.g. `2026-W28`),
because they are inherently time-based.

Templates for each file ship beside this skill in `references/` (`concept.md`, `session.md`,
`review.md`, `index.md`). Read the relevant template before writing so structure stays
consistent across machines and runs.

## Steps

### 1. Resolve LC_DIR, then sync the lessons repo

Use the Bash tool (cross-platform). **First resolve where the lessons repo lives** — it is
never assumed to be under home:

- If the `LEARNING_CENTER_DIR` environment variable is set, use it verbatim as `LC_DIR` and
  say nothing.
- If it is **unset**, ask the user where their learning-center repo is (or should be),
  offering `$HOME/learning-center` as the default they can accept with enter. Use their
  answer as `LC_DIR` for the rest of the run. Mention once that setting
  `LEARNING_CENTER_DIR` in their shell profile skips this prompt next time.

Confirm the variable and check whether it is set with the Bash tool, e.g.
`echo "${LEARNING_CENTER_DIR:-<unset>}"`, before deciding whether to ask.

Then, with `LC_DIR` set to the resolved path, ensure it exists and is current (do **not**
reintroduce a hardcoded `$HOME` path — use `"$LC_DIR"` throughout):

```
LC_DIR="<resolved path>"
if [ -d "$LC_DIR/.git" ]; then
  git -C "$LC_DIR" pull --ff-only
else
  git clone https://github.com/TGubbels/learning-center.git "$LC_DIR" \
    || { mkdir -p "$LC_DIR" && git -C "$LC_DIR" init \
         && git -C "$LC_DIR" remote add origin https://github.com/TGubbels/learning-center.git; }
fi
```

If the clone failed because the remote does not exist yet, the fallback inits a local repo;
tell the user to create `TGubbels/learning-center` on GitHub so the final push lands.

Compute the current week once and reuse it throughout: `WEEK=$(date +%G-W%V)` (e.g.
`2026-W28`). Use the environment's current date for session filenames.

_Done when:_ `LC_DIR` is resolved (from `LEARNING_CENTER_DIR` or the user's answer), exists as
a git repo and, if it had a remote, is pulled up to date, and you have `WEEK`.

### 2. Surface the concepts — or decide there are none

The source is always **the current conversation you are in** — no transcript hunting.

Extract only **genuine, transferable software-engineering / CS / AI concepts** — ideas that
generalize beyond this codebase and are worth learning as their own topic. Tag each **used**
(applied in the work) or **considered** (weighed, mentioned, or rejected — often the most
worth learning). Give each a canonical name and a kebab-case slug, and assign a **broader
category** (e.g. `concurrency`, `databases`, `http`, `testing`, `ai-llm`, `architecture`).

Keep the concept, drop the task: "fixed the N+1" surfaces **n-plus-one-query** and
**eager-vs-lazy-loading**.

**There is no quota — zero is a valid, common outcome.** Do not manufacture concepts to fill
a page. A session is worth a concept only if the idea would genuinely teach a competent
engineer something transferable. If nothing clears that bar, **skip to step 6 and write
nothing** — no files, no commit — reporting the one-line reason.

**A one-off, project-specific fix is not a concept.** If something learned is just a quirk
of *this* codebase — a config gotcha, a local API's surprising behaviour, a specific bug and
its fix — it does **not** get a concept file, and it does **not** get a standing reread-list
either. Its natural home is the **Example** section of a related concept. Only the rare
project fact that is *genuinely important to remember* and isn't a transferable concept earns
a single bullet under **Worth remembering** in the session recap (step 4).

_Done when:_ you have a set of named, slugged, categorized concepts (each tagged
used/considered) — or a clear, honest determination that the session produced none, in which
case go to step 6.

### 3. Write each concept as a standalone lesson

First read `LC_DIR/INDEX.md` (if present) to see which concepts already exist and how often
they have recurred, and pitch each lesson accordingly — deeper on the familiar/recurring,
more foundational on the brand-new.

Then, **resource-first**, use **WebSearch** for current, high-quality links (official docs,
MDN, respected deep-dives), confirm each points where you think, and write the lesson from
that material, **citing sources inline** with `[n]` markers that resolve in the Resources
section. When search is unavailable, give canonical resources from memory and mark them
`(from memory — verify)`.

For every concept from step 2, place its file at `LC_DIR/concepts/<category>/<slug>.md`
(loose at `LC_DIR/concepts/<slug>.md` only if it has no natural category), built from
`references/concept.md`, written so someone could learn it cold: YAML frontmatter, a one-line
TL;DR, an everyday analogy, the mental model, why it matters (with a key-insight callout), a
concrete example tied to *this* session (this is where a codebase specific belongs), the
tradeoffs (with a pitfall callout), verified resources cited inline, and the
retrieval-practice questions. Write for a tired human — lead with the plain idea, keep jargon
behind the analogy, and **keep it concise**: no filler, no restating.

**Frontmatter is the source of truth** for the INDEX and review file. Fill it fully:
`name`, `slug`, `category`, `status` (most recent used/considered), `first_seen`, `last_seen`,
`times`, and `weeks` (the list of every week the concept was touched).

**Diagrams — only when the concept is genuinely visual.** If a flow, sequence, structure, or
before/after would explain it faster than prose, add one **Mermaid** block (GitHub renders it
as a picture, no build step, stays diffable). If the concept is inherently textual (a naming
convention, a definition), skip it — don't force one. Keep diagrams simple and valid.

**Recurring concepts (the file already exists):** the path is stable, so there is nothing to
move. Open the existing file and merge into it — improve the teaching sections, append a
**terse one-line** entry under `## Session appearances`, and update the frontmatter:
bump `times`, set `last_seen` to today, append `$WEEK` to `weeks` (if not already present),
and refresh `status` to this session's tag. Find it with:

```
existing=$(find "$LC_DIR/concepts" -name "<slug>.md" 2>/dev/null | head -1)
```

_Done when:_ every concept from step 2 has a file at its stable category path (new ones
created, recurring ones merged in place with frontmatter bumped), all sections written and
concise, resources verified or flagged and cited inline, and Session appearances kept to one
terse line per visit.

### 4. Recap the session

Write `LC_DIR/sessions/$WEEK/YYYY-MM-DD-<short-topic>.md` from `references/session.md`: the
goal the session served, what it was about, the concepts surfaced (each linking to its
concept file at `../../concepts/<category>/<slug>.md`), key decisions, and what to study next.
Use the environment's current date.

In the **Concepts surfaced** table, mark each concept **new** or **updated** in the
`New / updated` column — a concept whose file you created fresh this run is **new**, one that
already existed and was merged is **updated**. For an updated concept, let the Note say what
this session added.

**Do not** write a standing "codebase-specific learnings" list. Codebase specifics live as
the Example inside a concept. Include the **Worth remembering** section *only* for the rare
project fact that is genuinely important and isn't a transferable concept — otherwise delete
that section entirely.

_Done when:_ the recap exists under the current week's folder, links to every concept file
from step 3, tags each concept new or updated, and carries a Worth-remembering bullet only if
one truly earned it.

### 5. Build the weekly review file, then update the index

**Review file** — write/update `LC_DIR/review/$WEEK.md` from `references/review.md`. It
collects the recall questions for **every concept touched during $WEEK** (derive that set
from the session recaps under `sessions/$WEEK/` — a recurring concept naturally reappears in
each week it was active, which is how spacing happens). For each question, include a
`<details>`-collapsed **concise** answer (enough to self-check; the full lesson stays in the
concept) and link the concept heading to `../concepts/<category>/<slug>.md`. Regenerate this
week's file wholesale so it reflects the current questions; leave earlier weeks' files
untouched.

**Index** — regenerate `LC_DIR/INDEX.md` (create from `references/index.md` if absent): group
concepts by category, one row per concept sourced from each file's frontmatter (`First seen`,
`Last seen`, `Times`, `Weeks`, `Used / considered`) with the stable `concepts/<category>/<slug>.md`
path. There is **no confidence column** — do not add one. Refresh **Suggested next to
study** (favour concepts that interleave with and space out what the KB already holds), the
**Review** list (link every `review/<week>.md`), and the **Sessions** list.

_Done when:_ `review/$WEEK.md` exists with this week's questions + collapsed answers + concept
links, and every concept in the KB has a current index row grouped by category with values
matching its frontmatter.

### 6. Commit locally, then ask before pushing

**If step 2 found no concepts:** write nothing, make no commit, and simply tell the user the
session produced nothing worth filing, with a one-line reason. Stop here.

Otherwise, always commit so the work is saved, but **never push without the user's
confirmation** — they review the generated files first.

```
git -C "$LC_DIR" add -A
git -C "$LC_DIR" commit -m "learning: <topic> — <n> concepts"
```

Then report what was filed — concepts (new vs. recurring), the session recap path, the review
file path, and the top suggested next study item — point the user at `LC_DIR` to review, and
**ask whether to push**. Only after they confirm:

```
git -C "$LC_DIR" push 2>/dev/null || echo "push skipped (no remote / not authed)"
```

If they decline, leave the commit in place and tell them they can push later (or re-run and
confirm) once they have reviewed.
