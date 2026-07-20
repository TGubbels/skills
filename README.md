# Skills repo

Personal Claude Code skills I use to develop myself as a software engineer.

## Skills

### `learning-summary`
Distills a coding/AI session into deep teaching notes and grows a personal concept
knowledge base. It surfaces the *transferable concepts beneath the tasks* (used **and**
considered), writes a concise, human-readable standalone lesson per concept (TL;DR, analogy,
mental model, example, tradeoffs, inline-cited resources, an optional Mermaid diagram, and
retrieval-practice questions), builds a weekly review file to study from, tracks your growth
in an index, and commits it.

- **User-invoked** (`/learning-summary`) — deliberate, side-effecting, zero context load.
- **Output goes to a dedicated lessons repo**, never the current project:
  `TGubbels/learning-center`, cloned to `~/learning-center` on any machine.
- Reads the current session you're in — no transcript hunting.
- **Concepts filed by category on a stable path** — `concepts/<category>/<slug>.md`, never
  moved; the week(s) a concept was touched live in its YAML frontmatter. Sessions and review
  files are filed by week (`sessions/<week>/`, `review/<week>.md`).
- **Study loop** — each week gets a `review/<week>.md` of that week's recall questions with
  collapsed answers, sized for a 5–10 minute retrieval burst.
- **No quota** — a session with nothing genuinely transferable produces nothing.
- Codebase-specific details live as a concept's *Example*, not a standing reread-list; the
  rare project fact worth remembering gets one line in the session recap.

## Installing (any machine, incl. WSL)

Via the [`skills`](https://github.com/vercel-labs/skills) CLI — the repo uses the standard
`skills/<name>/SKILL.md` layout, so no manifest is needed:

```sh
# one skill, globally, for Claude Code
npx skills@latest add TGubbels/skills --skill learning-summary --agent claude-code -g

# or every skill in this repo
npx skills@latest add TGubbels/skills -g
```

Manual alternative — symlink or copy the folder into your skills dir:

```sh
ln -s "$PWD/skills/learning-summary" ~/.claude/skills/learning-summary
```

Supporting files live in `references/` beside `SKILL.md` and travel with the skill on
install.

On first run the skill clones/pulls `~/learning-center` and commits lessons there — create
`TGubbels/learning-center` on GitHub so the push has somewhere to land.