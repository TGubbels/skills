---
name: learning-summary
description: Distill the current coding or AI session into deep, teachable notes and grow a personal concept knowledge base in your learning-center repo. Use at the end of a session to capture what was learned; commits locally and asks before pushing.
disable-model-invocation: true
---

Turn a working session into durable, teachable knowledge. The move that makes this skill
worth running: **surface the concept beneath the task** — file "exponential backoff", not
"added a retry loop". Do that every run and the knowledge base compounds.

Two principles from good teaching shape the output:

- **Resource-first** — find real sources before you explain; never teach a concept purely
  from parametric memory, and cite those sources *inline* in the explanation so every claim
  is traceable.
- **Storage over fluency** — write for long-term recall, not just a nice read: every
  concept file ends in retrieval-practice questions the user answers later.

## The lessons repo

All output goes to **one dedicated repo, never the current project**:

- **LC_DIR** — the local clone of the lessons repo. It is **not** hardcoded to home;
  resolve it at the start of every run (see step 1). It comes from the
  `LEARNING_CENTER_DIR` environment variable, and only falls back to
  `$HOME/learning-center` when that variable is unset.
- **LC_REMOTE** = `https://github.com/TGubbels/learning-center.git`

```
LC_DIR/
  INDEX.md                                    running overview + growth tracking
  concepts/<week>/<category>/<slug>.md         one standalone lesson per concept
  sessions/<week>/YYYY-MM-DD-<topic>.md        per-session recap linking to concepts
```

Everything is filed by **week** (`<week>` = ISO week, e.g. `2026-W28`) and concepts are
grouped into **broader categories** within each week. A concept file lives under the week it
was **last touched** — when it resurfaces in a later session it *moves* to the current
week's folder. Categories keep related concepts together (`concurrency`, `databases`,
`http`, `testing`, `ai-llm`, `architecture`, …); a concept with no natural category sits
loose directly under its week folder.

Templates for each file ship beside this skill in `references/`. Read the relevant template
before writing so structure stays consistent across machines and runs.

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

### 2. Surface the concepts

The source is always **the current conversation you are in** — no transcript hunting.

Extract only **genuine, transferable software-engineering / CS / AI concepts** — ideas that
generalize beyond this codebase and are worth learning as their own topic. Tag each **used**
(applied in the work) or **considered** (weighed, mentioned, or rejected — often the most
worth learning). Give each a canonical name and a kebab-case slug, and assign a **broader
category** (e.g. `concurrency`, `databases`, `http`, `testing`, `ai-llm`, `architecture`).

Keep the concept, drop the task: "fixed the N+1" surfaces **n-plus-one-query** and
**eager-vs-lazy-loading**.

**A one-off, project-specific fix is not a concept.** If something learned is just a quirk
of *this* codebase — a config gotcha, a local API's surprising behaviour, a specific bug and
its fix — it does **not** get a concept file. Capture it in the session recap's
**Codebase-specific learnings** section (step 4) instead. Only promote it to a concept if it
illustrates a transferable idea worth its own lesson.

_Done when:_ you have 3–8 named, slugged, categorized concepts (each tagged used/considered),
the trivial ones left out, the purely project-specific ones routed to the session recap, and
the genuinely new or subtle ideas kept.

### 3. Write each concept as a standalone lesson

First read `LC_DIR/INDEX.md` (if present) to see which concepts already exist and how often
they have recurred, and pitch each lesson accordingly — deeper on the familiar/recurring,
more foundational on the brand-new.

Then, **resource-first**, use **WebSearch** for current, high-quality links (official docs,
MDN, respected deep-dives), confirm each points where you think, and write the lesson from
that material, **citing sources inline** in the explanation with `[n]` markers that resolve
in the Resources section. When search is unavailable, give canonical resources from memory
and mark them `(from memory — verify)`.

For every concept from step 2, place its file at
`LC_DIR/concepts/$WEEK/<category>/<slug>.md` (loose at `LC_DIR/concepts/$WEEK/<slug>.md` only
if it has no natural category), built from `references/concept.md`, written so someone could
learn it cold: a one-line TL;DR, an everyday analogy, the mental model, why it matters (with a
key-insight callout), a concrete example tied to *this* session, the tradeoffs (with a pitfall
callout), verified resources cited inline, and the retrieval-practice questions. Write for a
tired human — lead with the plain idea, keep jargon behind the analogy.

**Diagrams — only when the concept is genuinely visual.** If a flow, sequence, structure, or
before/after would explain it faster than prose, add one **Mermaid** block (GitHub renders it
as a picture in the browser, no build step, and it stays diffable). If the concept is
inherently textual (a naming convention, a definition), skip the diagram — don't force one.
Keep diagrams simple and make sure the syntax is valid.

**Handle recurring concepts by moving them into the current week:**

```
existing=$(find "$LC_DIR/concepts" -name "<slug>.md" 2>/dev/null | head -1)
target="$LC_DIR/concepts/$WEEK/<category>/<slug>.md"
if [ -n "$existing" ] && [ "$existing" != "$target" ]; then
  mkdir -p "$(dirname "$target")"
  git -C "$LC_DIR" mv "$existing" "$target"
fi
```

When the file already existed, merge into the moved file: improve the teaching sections,
append a dated bullet under `## Session appearances`, and preserve `## My notes & links`
exactly as the user left it — that section is theirs. After a move, fix any inbound links to
the old path in `sessions/` recaps (grep the old relative path and repoint it). The INDEX is
regenerated wholesale in step 5, so it needs no manual patching.

**Housekeeping pass (every run, not just this session's concepts):** while you're here, tidy
the existing structure so grouping stays coherent as the KB grows —

- Any concept sitting **loose** directly under a week folder that clearly belongs to a
  category — especially one that now has two or more members — should be `git mv`d into that
  `<week>/<category>/` subfolder. Grouping beats loose files whenever a sensible category exists.
- Keep category slugs consistent: reuse an existing category name rather than coining a
  near-duplicate (`testing`, not a new `tests`). If you rename or consolidate a category
  folder, `git mv` every file into the canonical one.
- Do **not** move a concept to a different *week* here — a concept's week only changes when it
  actually resurfaces in the current session (the move block above). Housekeeping only fixes
  grouping *within* the week a file already lives in.

Repoint any inbound links you break with these moves (same grep-and-fix as above); the INDEX
is regenerated in step 5.

_Done when:_ every concept from step 2 has a file under the current week's folder (moved
there if it already existed elsewhere), loose concepts that fit a category have been grouped,
all sections written, resources verified or flagged and cited inline, pre-existing user notes
intact, and stale inbound links repointed.

### 4. Recap the session

Write `LC_DIR/sessions/$WEEK/YYYY-MM-DD-<short-topic>.md` from `references/session.md`: the
goal the session served, what it was about, the concepts surfaced (each linking to its
concept file), the **codebase-specific learnings** that didn't warrant a concept, key
decisions, and what to study next. Use the environment's current date.

In the **Concepts surfaced** table, mark each concept **new** or **updated** in the
`New / updated` column — you already know which from step 3: a concept whose file you created
fresh this run is **new**, one that already existed and was moved/merged into the current week
is **updated**. For an updated concept, let the Note say what this session added to the
existing lesson.

_Done when:_ the recap exists under the current week's folder, links to every concept file
from step 3, tags each concept new or updated, and captures the project-specific learnings
routed here in step 2.

### 5. Update the index

Regenerate `LC_DIR/INDEX.md` (create from `references/index.md` if absent): group concepts by
category, one row per concept, with current file paths (reflecting any moves), bumping `Last
seen`, `Times`, and `Last touched (week)` for any that recurred. There is **no confidence
column** — do not add one. Refresh **Suggested next to study** — favour concepts that
interleave with and space out what the KB already holds rather than more of the same.

_Done when:_ every concept in the KB has a current index row with the right path and week,
grouped by category, and the suggestions reflect this session.

### 6. Commit locally, then ask before pushing

Always commit so the work is saved, but **never push without the user's confirmation** —
they review the generated files first.

```
git -C "$LC_DIR" add -A
git -C "$LC_DIR" commit -m "learning: <topic> — <n> concepts"
```

Then report what was filed — concepts (new vs. recurring, and any that moved week/category),
the session recap path, and the top suggested next study item — point the user at `LC_DIR` to
review, and **ask whether to push**. Only after they confirm:

```
git -C "$LC_DIR" push 2>/dev/null || echo "push skipped (no remote / not authed)"
```

If they decline, leave the commit in place and tell them they can push later (or re-run and
confirm) once they have reviewed.
