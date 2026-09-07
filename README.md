# Zen for humans

The Zen of 80|20 for designing content, programs, screens, bundles of packages,
and whole systems.

Companion: [Zen for agents](AGENTS.md).

## 1. Don't do 20|80.

80|20 supports a ton of features with very little effort. Don't spend a ton of
effort on a very little feature.

## 2. Use before inventing.

Combine and reuse existing concepts first. When a new concept is necessary,
design it to serve more than the immediate use case and cooperate well with
existing concepts.

## 3. Do more with less.

Choose a small set of capabilities that delivers most of the practical value.
Simplicity should make the system useful, coherent, and dependable.

## 4. Give every responsibility a home.

Organize packages and bundles around meaningful areas of responsibility. Make
each part's purpose and its relationship to the whole easy to explain.

## 5. Prefer clarity over cleverness.

Make the important concepts and relationships easy to explain. Content,
programs, and screens should reinforce that understanding.

## 6. Leave room for creativity.

Use conventions to make ordinary work easy while leaving clear ways to adapt and
extend the design. Useful defaults should support different approaches as needs
evolve.

## 7. Keep dependencies deliberate.

Connect parts through what their responsibilities require. Allow each part to
evolve without forcing unrelated parts to change.

<details>
<summary>Background: source material and architecture review</summary>

## Basis for discussion

The strongest evidence of intent is the user's supplied material:

- Administration should be human oriented, connected, and built on reusable
  field types, structures, metadata, and help shared between DB and UUI.
- New concepts should combine across the system, remain simple, and leave room
  for customization. Verification should serve the actual problem.
- Development should isolate work, merge through Git, preserve running processes
  across activation and browser navigation, and keep recoverable private changes
  without expensive background scans.
- The kernel should be a stable foundation with clear boundaries. Packages
  should own changing system logic because they are easier to update.
- Repairs should follow ownership through the complete data flow and fix shared
  causes at their source.

Existing repository instructions and source inform the architecture review
below. They are evidence of the current design, not automatic endorsements of
it. The motto wording is an interpretation for discussion.

## Architecture review

Reviewed on 2026-09-06 against the sibling source worktrees. The most concrete
discrepancies concern development data and process lifetime. The broader
boundary issue is that some packages own schemas and command wrappers while
their policy remains compiled into Go. These are recommendations for discussion;
this review does not change platform behavior.

The separate [workspace architecture audit](../PLATFORM_REVIEW.md) extends this
review into dependency resolution, cross-node source distribution and startup
convergence, resource admission, transport bounds, and package compatibility. It
records its own experiments and their limits. Both reports are assessment
snapshots; the Zen above remains the proposed philosophy for discussion.

### The boundary to aim for

| Responsibility                                                                                                           | Owning layer                              |
| ------------------------------------------------------------------------------------------------------------------------ | ----------------------------------------- |
| Host processes, sandbox isolation, mounts, network listeners, routing, resource limits, private keys                     | Kernel                                    |
| Database connections, execution-scoped transactions, physical schema enforcement, safe host publication primitives       | Kernel                                    |
| Account eligibility, passwords, sessions, schedules, service declarations/defaults, development workflow, administration | Deno packages                             |
| Field meaning, help, domain relationships, screens, application session behavior                                         | Owning packages and shared Deno libraries |
| Published source identity, resolved runtime specifications, physical descriptors, execution identity                     | Explicit contracts between those owners   |

The deciding question is what requires kernel authority or must remain available
to recover the system. A feature being important, shared, or part of system
administration does not by itself answer that question. Equally, moving an
operation into Deno must preserve the kernel's ability to boot, isolate,
validate, and recover safely.

### 1. The later developer can overwrite an earlier conflicting edit

**High priority; demonstrated by an existing test.** `scanPackageChanges` takes
the current shared HEAD as the base of each scan. `preparePackage` uses
three-way Git machinery only if shared HEAD advances again after that capture.
It therefore lacks the original base of an already private edit when another
developer has published before the scan.
[Scan and preparation](../kernel/kernel/development/activation.go).

`TestActivationRebasesPrivateOverlayOnCurrentSharedSource` edits the same file
in two private sandboxes, activates A, then expects B to activate successfully
and replace the file with B's content. The test passed. That is the opposite of
the requested real-conflict behavior, despite the test calling it a rebase.
[Existing regression](../kernel/kernel/development/development_test.go).

The source/activation owner needs to retain the base of private divergence while
allowing untouched files to follow shared updates, then compare base, shared,
and private content through Git. Actual conflict content also needs to remain
available to the developer: the current cherry-pick path returns filenames and
an error, then deletes its temporary merge worktree. This is a shared overlay
and publication contract to repair, not a UUI conflict dialog workaround.

### 2. Successful publication destroys running development processes

**High priority; source and existing test agree.** `Activate` calls
`resetOverlayLocked`, which kills and deletes the sandbox before starting it
again. The terminal helper gets a special deferred reset after a 300 ms delay;
it still ends in the same destruction. Retaining a sandbox ID, home directory,
and installed tools does not retain its running processes.
[Activation](../kernel/kernel/development/activation.go),
[reset and delay](../kernel/kernel/development/overlay.go),
[helper response path](../kernel/kernel/development/manager.go).

`TestActivationScansOnlyOnDemandCommitsAndResetsOverlay` explicitly expects an
extra sandbox start after activation and passed. Publication and overlay
advancement need to preserve the running sandbox. The helper delay should
disappear when that owning lifecycle contract is corrected.
[Existing regression](../kernel/kernel/development/development_test.go).

### 3. A console connection owns the terminal's lifetime

**High priority; demonstrated by the WebSocket test.** The browser console is
disposed on navigation; the Go broker opens a PTY using the WebSocket request
context and closes it when the connection ends. The protocol opens a new console
rather than attaching to an independently identified running terminal.
`TestConsoleWebSocketStreamsAndResizes` requires the console to close with its
WebSocket and passed.
[Browser lifecycle](../uui/services/shell/frontend/custom_elements.ts),
[broker](../kernel/kernel/console/console.go),
[test](../kernel/kernel/console/console_test.go).

Give terminal execution an explicit owner and attach/detach lifetime, with
bounded output retention and explicit termination. Generic process/PTY mechanics
belong at the execution boundary; terminal selection and navigation belong in
packages. A server-side terminal owner inside the sandbox may also be
appropriate; evaluate that against existing capabilities before adding a second
process-management framework. Browser reconnect alone cannot preserve a process
whose server-side owner has closed it.

### 4. Checkpoint persistence does not establish recovery from sandbox loss

**High priority; source-established limitation, crash behavior not exercised.**
The live package overlay is disposable. Readable Git patches are saved at
explicit lifecycle boundaries, and normal stop refuses to proceed when capture
fails. However, manager shutdown calls stop/delete even after a checkpoint
error, and inherited-sandbox cleanup deletes the old filestore. The existing
persistence test proves orderly stop/start with a fake sandbox driver; it does
not prove recovery of edits after an unexpected sandbox loss.
[Checkpoint storage](../kernel/kernel/development/overlay.go),
[shutdown and inherited cleanup](../kernel/kernel/development/manager.go),
[current persistence test](../kernel/kernel/development/development_test.go).

There is also a source-visible capture window: activation captures its index
before pausing writers, then later replaces the overlay using that capture.
Writes accepted in between are at risk of being discarded. This race needs a
controlled runtime reproducer before claiming its frequency or exact impact.
[Capture ordering](../kernel/kernel/development/activation.go).

Define when a successful private write becomes recoverable and how publication
separates captured changes from later writes. Preserve recoverable state on
checkpoint failure and validate recovery after forced loss, including readable
reapplication elsewhere. The solution must satisfy this without an expensive
background scan; existing patch checkpoints are useful but insufficient proof of
that stronger contract.

### 5. A shared repository lock encloses slow and extensible work

**High priority for concurrency; demonstrated lock scope, no benchmark claim.**
Development activation pauses its sandbox, takes the shared repository mutex,
and retains it through Git preparation, schema evaluation, activation hooks,
publication, and overlay handling. Package repository mutation also holds that
shared mutex across staging and activation. Composition supplies the same mutex
to both managers.
[Development activation](../kernel/kernel/development/activation.go),
[repository mutation](../kernel/kernel/packages/repository.go),
[composition](../kernel/kernel/app/runtime.go).

Consequently, unrelated repository operations can wait behind filesystem,
subprocess, and Deno hook execution. This conflicts with the kernel's broad-lock
guidance and makes extension hooks part of contention behavior. Prepare work
outside broad locks and retain the necessary bounded publication/deployment
exclusion with revalidation of the expected source. Preserve atomicity and
recovery; merely deleting the lock would be an incorrect fix. Measure concurrent
developers and slow/reentrant hooks before claiming latency or deadlock safety.

### 6. Package-owned schemas can conceal kernel-owned policy

**Boundary discrepancy; some native authority is necessary.** The `packages`
repository owns table definitions and command programs, but Go implements
desired/active package records, activation phase history, hook checkpoints, and
publication. Development Go also chooses commit author defaults, commit-message
metadata, selection behavior, and the workflow around private changes. The
package command surface delegates to those implementations.
[Package contract](../packages/AGENTS.md),
[package records](../kernel/kernel/packages/package_index_database.go),
[activation coordinator](../kernel/kernel/packages/activation.go),
[development policy](../kernel/kernel/development/activation.go).

Schema residence alone therefore does not deliver independent policy updates.
Separate the minimum boot/recovery and host-publication contract from editable
development and package-management policy. Keep native mutations validated in
Go, while moving policy that can change independently to the owning packages.
Any records Go must read to boot become an explicit compatibility contract, even
if authored as package tables; they cannot be treated as freely changeable
application schemas. Resolve that bootstrap dependency before moving recovery or
activation orchestration wholesale.

### 7. The authentication transport fixes part of the session model in Go

**Boundary decision; not a demonstrated authentication failure.** Go token
verification requires `sid` and positive-integer `ver`, while Deno users creates
and interprets those session and account-version fields. Go also constructs
rejected-cookie deletion headers while users constructs application cookies.
These choices are explicitly documented today, so the question is whether that
is the intended long-term platform contract.
[Go token profile](../kernel/kernel/auth/token.go),
[users policy](../users/src/authentication.ts),
[current ownership contract](../kernel/kernel/auth/AGENTS.md).

Keep private keys, signature verification, trusted execution identity, and the
necessary transport envelope in Go. Decide explicitly whether session-specific
claim shape and cookie behavior are frozen protocol or package policy. If
packages should be free to change the session model, those requirements cannot
remain hidden kernel constraints. The fixed users module selected in
[composition](../kernel/kernel/app/authentication.go) is a default integration
point; its presence alone is not evidence that account logic lives in Go.

### 8. Program discovery has two eligibility contracts

**Confirmed source discrepancy; activation-window effects need runtime proof.**
The kernel lists and resolves programs from ready package records with active
commits and a bounded selector result. UUI Home instead rescans all mounted
program directories before each render, applies its own manifest parser, and
invokes the resulting filesystem entrypoint without checking package readiness.
Malformed manifests are skipped by the kernel listing but can abort UUI
discovery. These are materially different meanings of an available program.
[Kernel catalog](../kernel/kernel/packages/programs.go),
[UUI discovery and invocation](../uui/programs.ts),
[Home loop](../uui/programs/home/program.ts).

Define one authoritative contract for program availability and let UUI own
interactive invocation and presentation. Reuse or extend the existing catalog
contract, including an explicit development view if needed. Bound discovery work
and refresh it at meaningful source changes. The current UUI DOX specifically
asks for rescanning, so fixing this requires an agreed contract change rather
than labeling the implementation a violation of its local docs.
[Home contract](../uui/programs/home/AGENTS.md).

### A further release-contract question

Service versions record package commits and resolved policy, and job reuse
compares release IDs. Runtime sandboxes nevertheless mount the shared package
source, development activation mutates that source, and Workers import ordinary
entrypoint URLs. Metadata identifying a release does not by itself establish an
immutable filesystem or a complete dependency release.
[Version records](../services/src/indexing.ts),
[shared mount](../kernel/kernel/app/runtime.go),
[job identity](../kernel/kernel/execution/programs/programs.go),
[reuse comparison](../kernel/kernel/execution/jobs/jobs.go),
[module loading](../kernel/defaults/config/runtime/deno/worker/bootstrap.ts).

Test a running old-version service doing a delayed import or asset read across
activation, and a reused job after a dependency-only package update. Specify
which source and dependency version each execution must see. Mixed-source
behavior remains a source-derived risk in this review. The separate
[workspace audit](../PLATFORM_REVIEW.md) reports an actual RuntimeWorker
experiment in which an existing Worker retained the old dependency while a fresh
Worker loaded the new value; job reuse is opt-in. That narrower result does not
establish the behavior of a live service across activation. Settle both
contracts before claiming fully isolated release switching.

### Existing choices that support the philosophy

- Services resolves declarations, defaults, overrides, and effective versions in
  Deno, then publishes validated runtime specifications. This is a useful
  example of package policy with kernel enforcement.
  [Services contract](../services/AGENTS.md),
  [publication boundary](../kernel/kernel/app/reindex.go).
- Jobs owns calendars, durable occurrences, short claims, and history in Deno;
  generic kernel events and program execution supply the mechanism.
  [Scheduler](../jobs/src/runner.ts).
- Users owns password hashing and account/session validation, and UUI owns its
  protocol, replay, program recovery, and session metadata. These should remain
  package behavior. [Users contract](../users/AGENTS.md),
  [UUI contract](../uui/AGENTS.md).
- Deno DB owns authored descriptors, logical codecs, and Kysely integration; Go
  owns credentials, connection lifetime, and physical schema enforcement.
  Revalidating a physical descriptor at that authority boundary is appropriate.
  [DB contract](../db/AGENTS.md),
  [physical database contract](../kernel/kernel/database/AGENTS.md).
- The concurrent DB/UUI field work is moving toward shared semantic definitions.
  Keep callbacks and meaning in Deno and presentation in UUI; do not put help
  handlers into physical SQL descriptors or invent another package solely to
  rename the existing pure helper module. Unfinished work is not itself an
  architectural discrepancy. [Current field helpers](../db/fields.ts),
  [descriptor guidance](../db/src/AGENTS.md), [UUI adapter](../uui/fields.ts).
- The current trusted-package execution model is explicit in the kernel
  contract. Missing granular permissions are a declared phase limitation, not
  evidence that all existing package APIs violate their current trust model.
  [Execution contract](../kernel/kernel/AGENTS.md).

### Verification and recommended order

The review read both supplied texts, repository ownership guidance, and the
source paths above. Existing work in DB, UUI, and kernel was preserved. These
focused existing checks passed from the kernel repository:

```sh
.development/toolchains/go/bin/go test \
  ./kernel/console ./kernel/development ./kernel/packages \
  -run '^(TestConsoleWebSocketStreamsAndResizes|TestSandboxOverlayUsesSharedLowerAndPersistsCheckpoints|TestActivationScansOnlyOnDemandCommitsAndResetsOverlay|TestActivationRebasesPrivateOverlayOnCurrentSharedSource|TestActivationPublishesOnlyAfterBothHooks|TestPackagePublicationAndRevisionAreAtomic)$' \
  -count=1
```

The development checks use real Git with a fake sandbox driver; the console
check uses a real WebSocket with a fake console provider. They prove the tested
contracts, including the unwanted overwrite/reset/disconnect expectations. They
do not prove live gVisor behavior, browser usability, crash recovery, multi-node
operation, or performance. No such runtime or benchmark claims are made by this
review.

Discuss the principles first. For implementation, prioritize preservation and
correct publication of private work, then independent terminal lifetime and
bounded publication locks. Settle kernel/package compatibility and program
availability contracts before relocating code. Update the owning DOX and
regressions with each accepted behavior change, so successful tests and current
instructions support the desired system rather than the superseded behavior.

</details>
