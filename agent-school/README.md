# Agent School

![Agent School](./assets/agent-school-robot.png)

**Agent School** turns recurring agent mistakes into permanent, project-specific guidance. When agents keep getting something wrong—a library, a design pattern, or a convention—you use this skill to investigate the problem and generate a **Tessl tile** that teaches all future agents how to handle it correctly.

## What it does

The skill walks you through a structured, five-phase process:

1. **Interview** — Clarify what’s going wrong: concrete mistakes, examples, scope, and what “correct” looks like.
2. **Investigate** — Check existing tiles and rules, query library docs, read source code, and understand why agents get it wrong.
3. **Design** — Choose the right tile pieces (steering rules, docs, skills), name the tile, draft content, and get your approval.
4. **Create** — Add the tile directory, `tile.json`, and content (rules, docs, skills), then register it in `tessl.json`.
5. **Verify** — Confirm the tile is installed and that steering rules (if any) appear in `.tessl/RULES.md`.

The output is a **new tessl tile** checked into your repo: steering rules (always loaded), docs (on-demand), and/or skills (when relevant). That tile then steers every future agent session in that project.

## When to use it

Use Agent School when:

- Agents keep making the same mistakes around a library, API, or pattern.
- You want to codify project conventions or standards so agents follow them.
- You need repeatable, multi-step workflows (e.g. “how we add API endpoints”) captured as skills.

## Installation

Install via tessl:

```bash
tessl install jamesmoss/skills
```

**After you run Agent School:** the new tile it creates is part of your repo. Run `tessl install` in the project so steering rules are merged into `.tessl/RULES.md` and the new rules, docs, and skills become active.
