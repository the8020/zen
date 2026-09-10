# Zen for agents

The Zen of 80|20 for developing the platform and working with its internals.

Companion: [Zen for humans](README.md).

## 1. Don't do 20|80.

80|20 supports a ton of features with very little effort. Don't spend a ton of
effort on a very little feature.

## 2. Use before inventing.

Combine and reuse existing concepts first. When a new concept is necessary,
design it to serve more than the immediate use case and cooperate well with
existing concepts.

## 3. Respect boundaries.

Do not inject unrelated features into existing functionality. Package your
functionality as a standalone capability with clear responsibilities and
explicit dependencies. Integrate through the contracts of the components
involved.

## 4. The kernel is holy.

Touch the kernel only when absolutely necessary and with great care. No
application logic belongs there; it provides the foundation for the application
layer. Keep application behavior and policy in independently evolving Deno
packages.

## 5. Use the shared runtime.

Build on ordinary programs, services, jobs, hooks, and events using the existing
Worker runtime. Access kernel capabilities through the typed bridge and trusted
execution context. Extend shared mechanisms at their owner when necessary,
keeping them reusable across packages.

## 6. Share definitions across layers.

Compose ordinary Zod schemas and reuse their meaning across validation, database
tables, forms, and lists. Let shared database codecs handle physical
representations and the UUI framework handle browser presentation. Keep these
translations in their owning layers so application programs stay consistent and
small.

## 7. Give state and lifetimes clear owners.

Keep durable shared facts in the database and make node-local indexes and caches
explicitly derived. Distinguish connections, logical executions, Workers, and
sandboxes, with clear rules for completion, cancellation, and cleanup.
Reconnection or retry must respect the original execution’s identity and
outcome.

## 8. Bound work and resource use.

Keep queries, queues, retained output, background scans, and concurrency bounded
as the platform grows. Use targeted reads and updates, short transactions, and
explicit timeouts. Avoid holding broad locks across filesystem, process, or
network operations.

## 9. Fix and verify at the owner.

Read the applicable contracts and trace the full path before changing code.
Repair shared defects at their source, then verify both the owning layer and the
affected application flow. Keep documentation aligned with deliberate changes
across repository boundaries.

# Repository guidance

Parent DOX: [80|20 source workspace](../AGENTS.md).

Framework source:
[agent0ai/dox/AGENTS.md](https://github.com/agent0ai/dox/blob/765ae4ac02cc884eefcd41a3d0f71941721adb89/AGENTS.md).

# DOX framework

- DOX is highly performant AGENTS.md hierarchy installed here
- Agent must follow DOX instructions across any edits

## Core Contract

- AGENTS.md files are binding work contracts for their subtrees
- Work products, source materials, instructions, records, assets, and durable
  docs must stay understandable from the nearest applicable AGENTS.md plus every
  parent AGENTS.md above it

## Read Before Editing

1. Read the root AGENTS.md
2. Identify every file or folder you expect to touch
3. Walk from the repository root to each target path
4. Read every AGENTS.md found along each route
5. If a parent AGENTS.md lists a child AGENTS.md whose scope contains the path,
   read that child and continue from there
6. Use the nearest AGENTS.md as the local contract and parent docs for repo-wide
   rules
7. If docs conflict, the closer doc controls local work details, but no child
   doc may weaken DOX

Do not rely on memory. Re-read the applicable DOX chain in the current session
before editing.

## Update After Editing

Every meaningful change requires a DOX pass before the task is done.

Update the closest owning AGENTS.md when a change affects:

- purpose, scope, ownership, or responsibilities
- durable structure, contracts, workflows, or operating rules
- required inputs, outputs, permissions, constraints, side effects, or artifacts
- user preferences about behavior, communication, process, organization, or
  quality
- AGENTS.md creation, deletion, move, rename, or index contents

Update parent docs when parent-level structure, ownership, workflow, or child
index changes. Update child docs when parent changes alter local rules. Remove
stale or contradictory text immediately. Small edits that do not change behavior
or contracts may leave docs unchanged, but the DOX pass still must happen.

## Hierarchy

- Root AGENTS.md is the DOX rail: project-wide instructions, global preferences,
  durable workflow rules, and the top-level Child DOX Index
- Child AGENTS.md files own domain-specific instructions and their own Child DOX
  Index
- Each parent explains what its direct children cover and what stays owned by
  the parent
- The closer a doc is to the work, the more specific and practical it must be

## Child Doc Shape

- Create a child AGENTS.md when a folder becomes a durable boundary with its own
  purpose, rules, responsibilities, workflow, materials, or quality standards
- Work Guidance must reflect the current standards of the project or user
  instructions; if there are no specific standards or instructions yet, leave it
  empty
- Verification must reflect an existing check; if no verification framework
  exists yet, leave it empty and update it when one exists

Default section order:

- Purpose
- Ownership
- Local Contracts
- Work Guidance
- Verification
- Child DOX Index

## Style

- Keep docs concise, current, and operational
- Document stable contracts, not diary entries
- Put broad rules in parent docs and concrete details in child docs
- Prefer direct bullets with explicit names
- Do not duplicate rules across many files unless each scope needs a local
  version
- Delete stale notes instead of explaining history
- Trim obvious statements, repeated rules, misplaced detail, and warnings for
  risks that no longer exist

## Closeout

1. Re-check changed paths against the DOX chain
2. Update nearest owning docs and any affected parents or children
3. Refresh every affected Child DOX Index
4. Remove stale or contradictory text
5. Run existing verification when relevant
6. Report any docs intentionally left unchanged and why

## User Preferences

When the user requests a durable behavior change, record it here or in the
relevant child AGENTS.md

- Keep the complete upstream DOX framework in this workspace root and every
  repository root, and maintain the linked hierarchy in both directions.

## Child DOX Index

No child DOX documents. This document owns the entire repository.

# Maintaining the Zen

## Purpose

- Maintain two complementary statements of the 80|20 philosophy, grounded in the
  user's intentions.

## Ownership

- This independent `zen` repository contains exactly two authored files.
- `README.md` owns Zen for humans: higher-level design of content, programs,
  screens, package bundles, and whole systems.
- `AGENTS.md` owns Zen for agents: development and work with system internals,
  including ownership, contracts, implementation, resource costs, and
  verification. It also contains the full DOX framework and repository guidance.
- The parent link describes the sibling source workspace. This repository's
  framework and local instructions remain present in an independent checkout.

## Local Contracts

- The editions share principles and have distinct roles; they need not have
  matching wording or item counts. Maintain their consistency as either evolves.
- Use the user's instructions as the primary evidence of intent. Existing
  contracts explain current ownership; source and checks establish behavior.
- Keep these nine agent principles synchronized with the workspace root
  AGENTS.md. Reflect applicable rules in repository and domain Work Guidance;
  keep those local versions concrete and let covered descendants inherit them.
- Apply the Zen with the owning workspace and repository contracts. Report
  conflicts rather than treating an implementation accident as a principle.
- Keep the two-file structure and enduring principles. Completed reviews and
  temporary implementation reports do not belong in either edition.

## Work Guidance

- Write each Zen element as a short, understandable motto followed by one to
  three sentences explaining the thought and its practical implication.
- Keep the human edition general and abstract. Make the agent edition technical
  and grounded in the platform's languages, packages, execution model, and
  ownership boundaries; describe stable architectural rules rather than a
  particular feature or incident.
- Adapt the explanation to its audience and consolidate overlapping ideas. Use
  concrete examples when they clarify an enduring principle.

## Verification

- Apply the parent workspace's documentation checks: full framework, relative
  links, reciprocal parent/index links, and Markdown formatting.
- Confirm README.md and AGENTS.md are the only authored repository files and
  that the directory is an independent Git repository.
- Check both editions for their stated role, appropriate technical depth, and
  the motto plus one-to-three-sentence format. Keep implementation findings
  outside the Zen elements; this repository has no application test suite.
- Check that the workspace root carries the same nine agent principles and that
  affected domain guidance remains consistent with them.
