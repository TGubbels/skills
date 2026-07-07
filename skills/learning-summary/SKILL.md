---
name: learning-summary
description: Distill a coding/AI session into deep teaching notes and grow a personal concept knowledge base, committed to your learning-center repo. Run at the end of a session.
disable-model-invocation: true
---

Turn a working session into durable, teachable knowledge. The move that makes this skill
worth running: **surface the concept beneath the task** — file "exponential backoff", not
"added a retry loop". Do that every run and the knowledge base compounds.

Two principles from good teaching shape the output:

- **Resource-first** — find real sources before you explain; never teach a concept purely
  from parametric memory.
- **Storage over fluency** — write for long-term recall, not just a nice read: every
  concept file ends in retrieval-practice questions the user answers later.

## The lessons repo

All output goes to **one dedicated repo, never the current project**:

- **LC_DIR** = `$HOME/learning-center` (works on WSL, Linux, Mac, and Windows/Git-Bash).
- **LC_REMOTE** = `https://github.com/TGubbels/learning-center.git`

```
LC_DIR/
  INDEX.md                        running overview + growth tracking
  concepts/<slug>.md              one standalone lesson per concept
  sessions/YYYY-MM-DD-<topic>.md  per-session recap linking to concepts
```

Templates for each file ship beside this skill in `references/`. Read the relevant template
before writing so structure stays consistent across machines and runs.

## Steps

### 1. Sync the lessons repo

Use the Bash tool (cross-platform). Ensure `LC_DIR` exists and is current:

```
if [ -d "$HOME/learning-center/.git" ]; then
  git -C "$HOME/learning-center" pull --ff-only
else
  git clone https://github.com/TGubbels/learning-center.git "$HOME/learning-center" \
    || { mkdir -p "$HOME/learning-center" && git -C "$HOME/learning-center" init \
         && git -C "$HOME/learning-center" remote add origin https://github.com/TGubbels/learning-center.git; }
fi
```

If the clone failed because the remote does not exist yet, the fallback inits a local repo;
tell the user to create `TGubbels/learning-center` on GitHub so the final push lands.

_Done when:_ `LC_DIR` exists as a git repo and, if it had a remote, is pulled up to date.

### 2. Choose the source to analyze

- **Default:** the current conversation you are in.
- **A past/other session:** the user points you at one. Transcripts are JSONL under
  `~/.claude/projects/<project-slug>/*.jsonl`; list newest-first with Bash, and if which one
  is meant is ambiguous, ask before proceeding.

_Done when:_ you know exactly which conversation you are summarizing and have its content.

### 3. Surface the concepts

Extract the software-engineering and AI/CS concepts that appeared, each tagged **used**
(applied in the work) or **considered** (weighed, mentioned, or rejected — often the most
worth learning). Give each a canonical name and a kebab-case slug.

Keep the concept, drop the task: "fixed the N+1" surfaces **n-plus-one-query** and
**eager-vs-lazy-loading**. Rate each concept's confidence from how the session went — did
the user drive it, or need it explained?

_Done when:_ you have 3–8 named, slugged concepts, each tagged used/considered, with the
trivial ones left out and the genuinely new or subtle ones kept.

### 4. Write each concept as a standalone lesson

First read `LC_DIR/INDEX.md` (if present) to see the user's existing confidence per
concept, and pitch each lesson at the edge of what they already know — deeper on the
familiar, more foundational on the new.

Then, **resource-first**, use **WebSearch** for current, high-quality links (official docs,
MDN, respected deep-dives), confirm each points where you think, and write the lesson from
that material. When search is unavailable, give canonical resources from memory and mark
them `(from memory — verify)`.

For every concept from step 3, create or update `LC_DIR/concepts/<slug>.md` from
`references/concept.md`, written so someone could learn it cold: the mental model, why it
matters, a concrete example tied to *this* session, the tradeoffs, 2–4 verified resources,
and the retrieval-practice questions.

When the file already exists, merge: improve the teaching sections, append a dated bullet
under `## Session appearances`, and preserve `## My notes & links` exactly as the user left
it — that section is theirs.

_Done when:_ every concept from step 3 has a file with all sections written, resources
verified or flagged, and any pre-existing user notes intact.

### 5. Recap the session

Write `LC_DIR/sessions/YYYY-MM-DD-<short-topic>.md` from `references/session.md`: the goal
the session served, what it was about, the concepts surfaced (each linking to its concept
file), key decisions, and what to study next. Use the environment's current date.

_Done when:_ the recap exists and links to every concept file from step 4.

### 6. Update the index

Update `LC_DIR/INDEX.md` (create from `references/index.md` if absent): one row per concept,
bumping `Last seen` and `Times seen` for any that recurred, and refresh **Suggested next to
study** — favour concepts that interleave with and space out what the KB already holds
rather than more of the same.

_Done when:_ every concept in the KB has a current index row and the suggestions reflect
this session.

### 7. Commit and push

```
git -C "$HOME/learning-center" add -A
git -C "$HOME/learning-center" commit -m "learning: <topic> — <n> concepts"
git -C "$HOME/learning-center" push 2>/dev/null || echo "push skipped (no remote / not authed)"
```

Then report: concepts filed, new vs. recurring, the top suggested next study item, and
whether the push succeeded.