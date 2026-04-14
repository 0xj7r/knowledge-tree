---
name: project-knowledge-tree
description: Use when the user wants to build up or maintain a living, tree-structured knowledge base about a project - architecture decisions, integration contracts, open questions, glossary - so context accumulates across sessions instead of being lost. Trigger on phrases like "add this to the knowledge base", "document this decision", "write up the architecture", "what do we know about X", or when the user signs off on a non-trivial design choice that should outlive this conversation.
---

# Project Knowledge Tree

## Overview

Maintain a `docs/knowledge/` directory in the project as a tree of markdown nodes. Every node has one stable purpose. New knowledge is appended or added as a new node. Existing nodes are updated in place with a timestamp — never silently rewritten. README.md index files in each subdir are regenerated automatically after changes.

## When to use

- User signs off on a design decision ("ok this makes sense", "let's go with that").
- User says "add this to the knowledge base" or "document X".
- User asks "what did we decide about X" or "what do we know about Y".
- Starting a conversation where the canonical current state of the project matters.

## Tree shape

```
docs/knowledge/
├── README.md                   # top-level index (regenerated)
├── architecture/               # one file per architectural unit
│   ├── README.md
│   └── <component>.md
├── decisions/                  # append-only log
│   ├── README.md
│   └── YYYY-MM-DD-<topic>.md
├── integrations/               # one file per vendor/service
│   ├── README.md
│   └── <name>.md
├── contracts/                  # schemas, envelopes, API shapes
│   ├── README.md
│   └── <contract>.md
├── open-questions/             # running list with status
│   └── README.md
└── glossary.md                 # canonical terminology
```

Only create subdirs when you have a node to put in them.

## Node template

Every node starts with:

```yaml
---
title: <human-readable name>
status: draft | approved | stale
last_updated: YYYY-MM-DD
related: [path/to/other-node.md]
---
```

Body sections by node type:
- **architecture/** — Purpose, Owners, Components, Key contracts, Open questions, Decision log
- **decisions/** — Context, Decision, Rationale, Consequences, Follow-ups
- **integrations/** — Purpose, Endpoints, Auth, Quirks, Related decisions
- **contracts/** — Shape (with code block), Invariants, Versioning
- **open-questions/** — Question, Context, Options, Who decides, Status

Templates for each are in `templates/`.

## Workflow

**Adding a signed-off decision:**
1. Identify the right architecture node (existing or new). Search the tree before assuming it's new.
2. If new node: copy the template, fill in, set `status: approved`, `last_updated: today`.
3. If existing node: edit in place, bump `last_updated`, append to its Decision log section.
4. Log the decision separately at `decisions/YYYY-MM-DD-<slug>.md` if it's substantive (worth a standalone entry in a month).
5. If the decision resolves an open question, edit that entry in `open-questions/README.md` — mark resolved, link to the decision.
6. Regenerate affected `README.md` indexes.

**Updating existing knowledge:**
1. Re-read the node(s) first. Don't trust your memory from earlier in the session.
2. Append or modify sections; preserve prior context. If the change invalidates prior content, mark the old section as `> Superseded YYYY-MM-DD — see <reason>` rather than deleting.
3. Bump `last_updated`.
4. If the node was `approved` and the update represents a material change, flip to `draft` until the user signs off again.

**Answering "what do we know about X":**
1. Start at `docs/knowledge/README.md`.
2. Follow into the subdir, then the specific node.
3. Pull in `related` nodes from frontmatter when relevant.
4. If the answer isn't there, say so — don't fabricate.

## Index regeneration

After any add/remove in a subdir, regenerate its `README.md` as a table:

```markdown
| Title | Status | Last updated | Link |
|---|---|---|---|
| Platform service | approved | 2026-04-16 | [platform.md](platform.md) |
| Data plane | approved | 2026-04-16 | [data-plane.md](data-plane.md) |
```

Top-level `README.md` lists each subdir with a one-line description plus its node count.

Parse frontmatter (`title`, `status`, `last_updated`) from each `.md` file to build the table. Skip README.md itself.

## Status meanings

- `draft` — in progress, not yet signed off.
- `approved` — user-signed-off, canonical. Treat as load-bearing.
- `stale` — known to be outdated; should be refreshed or superseded. Never silently rewritten.

If observed reality contradicts an `approved` node, flip it to `stale` and propose the update — don't rewrite history.

## Common mistakes

- **Rewriting instead of appending.** Preserve context; timestamp changes.
- **Creating duplicate nodes.** Search the tree before assuming it's new.
- **Forgetting the index.** An un-indexed node is effectively lost.
- **Dumping raw conversation.** Distill into structured sections — nodes are navigable, not transcripts.
- **Conflating decision logs with architecture nodes.** Decisions record *what changed and why*. Architecture nodes describe *current state*.
- **Regenerating index from guesswork.** Always parse frontmatter from actual files.

## Red flags

- About to rewrite a node instead of append → stop, mark superseded.
- About to create a new node without searching → stop, grep the tree first.
- About to skip the index regen → stop, the node won't be findable.
- About to write prose without frontmatter → stop, every node needs the header.

## Verification

Healthy tree satisfies:
1. Every node has valid frontmatter.
2. Every README.md lists every sibling `.md` file.
3. Every open-question has either a linked decision or a current status.
4. Every term in node titles is defined in `glossary.md` (or obvious).
5. `last_updated` within 90 days OR `status: stale`.

Run `ls docs/knowledge/**/*.md` and eyeball the tree quarterly.
