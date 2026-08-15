# Dev Stack Template Library

Stack-specific implementation patterns and code generation guides for Claude Code.
Each file is a companion to the matching SA stack template — Dev copies or references it as project-level knowledge to prevent Claude Code from generating code that deviates from the agreed architecture.

---

## Relationship to SA Stack Templates

SA owns `STACK_CONTEXT.md` — the stack decisions document that travels SA → PO → Dev.
Dev templates in this directory contain the **implementation detail** that does not belong in `STACK_CONTEXT.md`: bootstrap steps, layer-by-layer code examples, anti-patterns, and guardrails that prevent Claude Code from violating architecture boundaries.

Both files work together:

| File | Owned by | Contains |
|------|----------|----------|
| `STACK_CONTEXT.md` | SA | Stack decisions — what and why |
| Dev template (this directory) | SA/Lead hands off to Dev | How to implement correctly |

---

## How to use

### SA / Lead

When SA selects a stack and fills in `STACK_CONTEXT.md`, note the companion Dev template path in the Lead Handoff so Dev knows where to find it.

### Dev

1. Find the template matching your project's stack (see table below)
2. Copy it into the project as supplementary knowledge — attach to Claude Code project knowledge, or append below the project-specific section of `CLAUDE.md`
3. Do not modify the template sections — add project-specific notes under a `## Project Overrides` heading at the bottom

---

## Available Templates

| Template | Stack | Companion SA template |
|----------|-------|-----------------------|
| [node-fastify/DEV_PATTERNS.md](node-fastify/DEV_PATTERNS.md) | Node.js + Fastify v5 + Prisma v7 + 5-layer Clean Architecture | `STACK_CONTEXT_node_fastify.md` |
| _(add more as stacks are adopted)_ | — | — |

**Rule:** do not create Dev templates for hypothetical stacks — only when a real project in the organisation uses that stack and the SA template already exists.
