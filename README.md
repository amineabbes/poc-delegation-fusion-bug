# POC: delegated writer file-editing behavior in Copilot Fusion

This POC investigates differences in file-editing behavior across
requested child models when using parent/child agent delegation in
Copilot Fusion in VS Code.

The main conversation remains in its Fusion configuration and acts only
as the orchestrator. Each test launches a fresh parent with an explicitly
requested model, which then invokes the same custom `writer` agent with
an explicitly requested child model.

The goal is to provide a reproducible support case in this execution
context, not to establish that Fusion or model heterogeneity causes
the observed behavior.

## Start here

- [Full test report](report.md)
- [Results matrix](report.md#results-matrix)
- [Per-test evidence](report.md#per-test-evidence)
- [Declared tool inventories](report.md#declared-inventories)
- [Trace limitations and conclusion](report.md#trace-limitations-and-conclusion)
- [Reproduction prompt](prompt.md)
- [Custom writer agent](.github/agents/writer.agent.md)

The report is the reference for environment details, individual outcomes,
inventory declarations, and evidence limitations.

## Context and test architecture

The tests run from a main Copilot conversation using Fusion in VS Code:

```text
Copilot Fusion main conversation / orchestrator
└── Fresh parent with an explicitly requested model
    └── Fresh custom writer child with an explicitly requested model
        └── Write one exact marker to a dedicated result file
```

Fusion is the main conversation's configuration, not the model identity
of the tested parents or children. Those models are explicitly requested
for each delegation.

The custom `writer` agent is configured with the tool capabilities
`read`, `edit`, and `search`. The same agent definition is used unchanged
throughout the run.

Eight selected parent/child combinations cover both within-family and
cross-family delegation. Every test uses the same child prompt template,
changing only the target path and marker.

Only the assigned writer child may write its result file. Terminal,
shell, script, and other writing workarounds are prohibited. Unsuccessful
attempts are not repaired or retried within a run.

## Results overview

The latest reported run produced the following outcomes:

| Parent \ Child | gpt-6-luna | gpt-6.1-terra | claude-opus-5.5 | claude-fable-5.2 |
|---|---|---|---|---|
| gpt-6-luna | — | CONTENT VALID (T01) | FILE ABSENT (T03) | FILE ABSENT (T04) |
| gpt-6.1-terra | — | — | FILE ABSENT (T05) | — |
| claude-opus-5.5 | CONTENT VALID (T06) | CONTENT VALID (T08) | — | FILE ABSENT (T02) |
| claude-fable-5.2 | CONTENT VALID (T07) | — | — | — |

Model names identify the requested combinations, subject to the runtime
verification limits described below.

- `CONTENT VALID`: the file contains exactly the expected marker,
  optionally followed by one LF byte.
- `FILE ABSENT`: the expected result file does not exist.
- `—`: the combination was not selected for testing.

These are file outcomes, not independent confirmation of the complete
nested execution.

See the [per-test evidence](report.md#per-test-evidence) for the associated
declarations, operation records, and verification limits.

## Observed tool declarations

All eight returned writer responses include a child inventory declaration.
Parent and child inventory declarations are recorded separately.

| Requested child models | Returned child inventory | File outcomes |
|---|---|---|
| `gpt-6-luna`, `gpt-6.1-terra` | `functions.view`, `functions.rg`, `functions.glob`, `functions.apply_patch`, `multi_tool_use.parallel` | Exact expected content in T01, T06, T07, T08 |
| `claude-opus-5.5`, `claude-fable-5.2` | `view`, `grep`, `glob` | File absent in T02, T03, T04, T05; no editing call reported |

The writer definition's SHA-256 was unchanged before and after the tests.

The observed pattern is not simply a failure of cross-family delegation:
the tested Claude-parent → GPT-child combinations produced valid files,
while the Claude-parent → Claude-child combination T02 did not.

In this run, the file outcomes and returned inventory declarations differ
according to the requested child model family. This association does not
establish a root cause or justify generalizing to untested combinations.

See the [declared inventories](report.md#declared-inventories) for the
parent and child lists.

## Expected behavior

The custom writer agent's configured edit capability should be available
through an appropriate file-editing tool for each supported selected child
model, unless an explicit restriction or capability limitation applies.

Concrete tool names may differ between models. The expectation is not
identical inventories, but the ability to perform the same permitted
file-writing task with the same writer profile.

If a selected combination does not support editing, that limitation
should be surfaced explicitly rather than leaving the writer with only
read/search tools and no clear explanation.

## Evidence and limitations

The report distinguishes:

- Independently checked file existence and final bytes.
- Parent and child inventory declarations.
- Editing operations reported in returned writer responses.
- Available local operation records.
- Runtime information that could not be independently verified.

The local `session_files` index records `apply_patch` operations for the
four written targets. These entries corroborate operations on those
paths, but do not, by themselves, establish the runtime identity of the
writer child.

The orchestrator explicitly selected the requested parent models in its
launch arguments. Accessible records did not expose the nested child-launch
arguments or child call IDs. Consequently, the exact child model, agent
selection, and full child prompt could not be independently confirmed
from runtime arguments.

Declared inventories are not independently verified runtime inventories.
A missing file does not, by itself, prove that an editing tool was absent.

No execution deviation was identified in the available evidence.
This is not equivalent to independent verification of all nested
execution details.

See the [trace limitations and conclusion](report.md#trace-limitations-and-conclusion).

## Reproduction

1. Use a workspace containing the supplied
   [writer definition](.github/agents/writer.agent.md).
2. Preserve that definition unchanged, including its configured
   `read`, `edit`, and `search` capabilities.
3. Ensure the result paths listed in [prompt.md](prompt.md) are absent.
   Prefer a fresh workspace copy for a new run; preserve existing evidence.
4. In VS Code, use a main Copilot conversation configured with Fusion.
   Execute the complete contents of [prompt.md](prompt.md), keeping the
   main conversation in that configuration. The prompt explicitly
   selects the requested parent and child models for each test.
5. Review the generated `report.md` and retain the result files and
   accessible traces.

The reproduction prompt defines the model mapping, eight test cases,
execution rules, response-preservation requirements, and validation
procedure.

Each writer is instructed to include its own declared inventory in its
final response, even if it already reported that inventory in a progress
message. Each parent must preserve the complete returned writer response
separately from its own inventory. This avoids relying on progress messages
being included in the returned task response.

If a configured result path already exists, the protocol skips that test
rather than deleting or overwriting the file. Preserve the supplied report
before rerunning, because a new execution writes `report.md`.

## Requested investigation

Please investigate whether the configured `edit` capability is exposed
correctly to delegated writer children across the requested models in
this Copilot Fusion execution context.

In particular:

- Compare the concrete tools exposed to the writer children.
- Verify the actual nested model and custom-agent selections.
- Verify that the intended child prompt was transmitted unchanged.
- Determine whether a capability restriction or tool-exposure issue
  explains the observed difference.

These checks may require runtime records unavailable in the accessible
POC traces.

The supplied report documents the observed behavior and its evidence
limits. It does not establish that Fusion, the parent model, or model
heterogeneity caused the difference.