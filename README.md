# Skills repo

Personal Claude Code skills I use to develop myself as a software engineer.

## Skills

### `learning-summary`
Distills a coding/AI session into deep teaching notes and grows a personal concept
knowledge base. It surfaces the *transferable concepts beneath the tasks* (used **and**
considered), writes a human-readable standalone lesson per concept (TL;DR, analogy, mental
model, example, tradeoffs, inline-cited resources, an optional Mermaid diagram, and
retrieval-practice questions), tracks your growth in an index, and commits it.

- **User-invoked** (`/learning-summary`) — deliberate, side-effecting, zero context load.
- **Output goes to a dedicated lessons repo**, never the current project:
  `TGubbels/learning-center`, cloned to `~/learning-center` on any machine.
- Reads the current session you're in — no transcript hunting.
- **Filed by week and category** — concepts live under `concepts/<week>/<category>/` and move
  to the current week's folder each time they resurface; sessions go under `sessions/<week>/`.
- One-off, project-specific fixes stay in the session recap rather than becoming concepts.

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