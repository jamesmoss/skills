---
name: agent-school
description: Investigates a problem area in the codebase and generates a new tessl tile (rules, docs, skills) to teach agents how to handle it correctly. Use when agents keep making the same mistakes around a library, design pattern, or convention.
---

# Tile from Codebase

You are a tile author. Your job is to investigate a problem area in this codebase and produce a new tessl tile that permanently teaches agents how to handle it correctly.

A tessl tile can contain any combination of:

- **Steering rules** — loaded eagerly into every agent session. Best for conventions, patterns, and standards agents must always follow.
- **Docs** — loaded on-demand when agents query for library information. Best for API references, version-specific behavior, and usage guides.
- **Skills** — loaded when relevant to the user's request. Best for repeatable multi-step workflows.

Your output is not an explanation to the user. Your output is a **new tile** checked into the repo that will steer all future agents.

## Phase 1: Interview the User

Start by understanding what the user is experiencing. Parse their initial prompt, then ask clarifying questions. You need to understand:

1. **What specific mistakes are agents making?** Get concrete examples if possible — wrong function calls, incorrect patterns, misused APIs, etc.
2. **Can they point to examples?** Ask for file paths showing the correct approach, or descriptions of the wrong behavior they've seen.
3. **What is the scope?** Is this problem specific to one app (`apps/backend`, `apps/frontend`, `apps/atlas`) or project-wide?
4. **What does "correct" look like?** How should agents handle this area? Are there existing examples of the right approach in the codebase?

Do not skip this phase. Ask your questions, wait for answers, then proceed.

## Phase 2: Investigate the Codebase

With the user's answers in hand, investigate the codebase to build a complete picture.

### 2a: Check Existing Tiles and Rules

First, check if this area is already covered by an existing tile:

- Read the root `tessl.json` and any app-level `tessl.json` files
- Read `.tessl/RULES.md` and app-level `.tessl/RULES.md` files
- Search the `tiles/` directories for related content

If an existing tile partially covers this area, you may want to extend it rather than create a new one. Ask the user.

### 2b: Query Library Documentation

Use the `query_library_docs` MCP tool to search for any existing documentation relevant to the problem area. Try multiple queries:

- The library or framework name
- The specific API or pattern involved
- Error messages agents might encounter

### 2c: Read Source Code

Search the codebase for patterns related to the problem:

- Find examples of the **correct** approach (use Grep and Glob to locate them)
- Find examples of the **incorrect** approach if they exist
- Read configuration files (`package.json`, `tsconfig.json`, etc.) for version information
- Read any relevant README or documentation files

### 2d: Understand the "Why"

Understand why agents get this wrong. Common reasons:

- The library version in this project has different APIs than what agents were trained on
- The project uses a non-standard pattern that conflicts with common conventions
- There are project-specific constraints that aren't documented anywhere
- The correct approach requires multiple steps that aren't obvious

## Phase 3: Design the Tile

Based on your investigation, decide what the tile should contain.

### Choosing Tile Components

| Problem Type | Recommended Components |
|---|---|
| Agents use wrong API / outdated patterns for a library | **Docs** (primary) + **Rules** (key constraints) |
| Agents don't follow project conventions or standards | **Rules** (primary) |
| Agents don't follow a multi-step process correctly | **Skills** (primary) + **Rules** (key constraints) |
| Agents misuse an internal utility or abstraction | **Docs** (primary) + **Rules** (key constraints) |
| Complex area needing both reference material and guardrails | **Docs** + **Rules** |

### Choosing a Name

Pick a clear, descriptive tile name. Use the namespace appropriate for the scope:

- `workspace/<name>` — project-wide tiles (registered in root `tessl.json`)
- `<app-name>/<name>` — app-specific tiles (registered in `apps/<app>/tessl.json`, files in `apps/<app>/tiles/<name>/`)

Name examples: `workspace/drizzle-migrations`, `tessl-backend/result-pattern`, `workspace/api-endpoint-workflow`.

### Drafting Content

For each component, draft the content:

**Steering Rules** should include:
- A clear statement of what agents must do and must not do
- Code examples from the actual codebase showing the correct pattern
- Code examples showing the incorrect pattern (what to avoid)
- Brief explanation of why this matters

**Docs** should include:
- Accurate API reference for the specific version used in this project
- Usage examples pulled from or based on real code in the codebase
- Common pitfalls and how to avoid them
- Links to related files in the codebase

**Skills** should include:
- Step-by-step phases the agent should follow
- What tools to use at each step (Grep, Glob, Read, query_library_docs, etc.)
- Verification steps to confirm the work is correct
- Examples of expected output

### Confirm with the User

Before creating files, present your plan to the user:

- The tile name
- Which components it will include (rules, docs, skills)
- A summary of what each component will cover
- Where the tile will be registered (root or app-level `tessl.json`)

Wait for user approval before proceeding to Phase 4.

## Phase 4: Create the Tile

### 4a: Create the Directory Structure

Create the tile directory. The structure depends on which components you're including:

**Rules only:**
```
tiles/<tile-name>/
  tile.json
  steering/
    rules.md
```

**Docs only:**
```
tiles/<tile-name>/
  tile.json
  docs/
    index.md
```

**Skills only:**
```
tiles/<tile-name>/
  tile.json
  skills/<skill-name>/
    SKILL.md
```

**Combined (use only what's needed):**
```
tiles/<tile-name>/
  tile.json
  steering/
    rules.md
  docs/
    index.md
  skills/<skill-name>/
    SKILL.md
```

### 4b: Write tile.json

The `tile.json` format depends on which components are included:

```json
{
  "name": "<namespace>/<tile-name>",
  "version": "0.0.1",
  "summary": "<one-line description>",
  "steering": {
    "<steering-key>": {
      "rules": "steering/rules.md"
    }
  },
  "docs": "docs/index.md",
  "skills": {
    "<skill-name>": {
      "path": "skills/<skill-name>/SKILL.md"
    }
  }
}
```

Only include the `steering`, `docs`, and `skills` fields that are relevant. Do not include empty fields.

### 4c: Write the Content Files

Write the drafted content from Phase 3 into the appropriate files. Make sure to:

- Use real file paths and code examples from the codebase (not fabricated examples)
- Keep steering rules concise and actionable — agents read these on every session
- Keep docs thorough but organized — agents pull these on demand
- Keep skills structured with clear phases — agents follow these step by step

For SKILL.md files, use YAML front matter:

```markdown
---
name: <skill-name>
description: <when to use this skill>
---

# <Skill Title>

<instructions>
```

### 4d: Register the Tile

Add the tile as a dependency in the appropriate `tessl.json`:

- For project-wide tiles: edit the root `tessl.json`
- For app-specific tiles: edit `apps/<app>/tessl.json`

Add an entry like:
```json
"<namespace>/<tile-name>": { "source": "file:tiles/<tile-name>" }
```

### 4e: Install

Run `tessl install` using the MCP tool to wire up the new tile. This regenerates `.tessl/RULES.md` and makes the tile active.

## Phase 5: Verify

1. Run `tessl status` to confirm the tile is recognized and installed
2. Check the relevant `.tessl/RULES.md` to confirm steering rules appear (if applicable)
3. Run `bun lint` to check for any issues
4. Tell the user the tile is ready and summarize what it contains

## Important Notes

- Always use **real code examples** from the codebase, not generic examples
- Keep steering rules **concise** — they're loaded into every agent session and consume context
- Prefer **extending an existing tile** over creating a new one if the area is already partially covered
- If the problem is very broad, consider creating **multiple focused tiles** rather than one large one
- The tile should be **self-contained** — an agent reading it should understand the rules without needing to ask follow-up questions
