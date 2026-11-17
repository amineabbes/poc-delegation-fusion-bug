# Reproduce file-editing behavior across delegated models

Run the eight tests below in the current workspace and write a concise
English report to `report.md`.

The purpose is to observe whether a custom writer child can edit files
under different parent/child model combinations. Do not assume which
combinations should succeed.

## Setup

Use the existing `.github/agents/writer.agent.md` unchanged.
Its configured tools must include `read`, `edit`, and `search`.

Keep the main conversation on its current configuration. It is the
orchestrator, not a tested parent.

Read the writer definition and record its SHA-256.
Ensure `results/` exists and all eight target paths below are absent.
If any target already exists, stop and request a clean starting state.
Do not delete existing results.

Record readily available VS Code, Copilot, and OS versions.
Do not investigate unavailable environment information.

## Test plan

| Test | Parent model | Writer child model | Target path | Marker |
|---|---|---|---|---|
| T01 | gpt-6-luna | gpt-6.1-terra | results/t01-gpt-a-to-gpt-b.txt | T01_GPT_A_TO_GPT_B |
| T02 | claude-opus-5.5 | claude-fable-5.2 | results/t02-cld-c-to-cld-d.txt | T02_CLD_C_TO_CLD_D |
| T03 | gpt-6-luna | claude-opus-5.5 | results/t03-gpt-a-to-cld-c.txt | T03_GPT_A_TO_CLD_C |
| T04 | gpt-6-luna | claude-fable-5.2 | results/t04-gpt-a-to-cld-d.txt | T04_GPT_A_TO_CLD_D |
| T05 | gpt-6.1-terra | claude-opus-5.5 | results/t05-gpt-b-to-cld-c.txt | T05_GPT_B_TO_CLD_C |
| T06 | claude-opus-5.5 | gpt-6-luna | results/t06-cld-c-to-gpt-a.txt | T06_CLD_C_TO_GPT_A |
| T07 | claude-fable-5.2 | gpt-6-luna | results/t07-cld-d-to-gpt-a.txt | T07_CLD_D_TO_GPT_A |
| T08 | claude-opus-5.5 | gpt-6.1-terra | results/t08-cld-c-to-gpt-b.txt | T08_CLD_C_TO_GPT_B |

## Execution

Execute T01 through T08 sequentially. Each test is independent.
A failed writing task does not stop the remaining tests.

For each test:

1. Render the child prompt below using the exact path and marker from
   the table. Do not shorten either value.

2. Launch one fresh general-purpose parent through `task`, explicitly
   selecting the parent model. Put the parent instructions in the
   launcher's `prompt` argument, not in an argument named `task`.

3. Give that parent only:
   - its test ID;
   - the requested child model;
   - the parent instructions below;
   - the fully rendered child prompt as a JSON string value, clearly
     labelled `CHILD_PROMPT`.

   The JSON string must contain the actual complete text, not a
   placeholder or a reference to another section.

4. Wait for the parent to finish, then independently inspect the target
   file without modifying it.

At both delegation levels, use the actual tool schema and include all
required fields, including description, name, agent_type, and prompt
where required. Select models explicitly. Use `mode: sync` when supported;
otherwise use the tool's documented completion behavior or wait mechanism.
Do not send unsupported fields.

Do not switch or substitute models, reuse agents, retry a test, or send
follow-up instructions to a completed child.

If a test cannot be launched correctly, record the blocker and continue
with the next test where possible.

## Parent instructions

You are the fresh parent for the assigned test.

Before delegating, list the exact tool identifiers visible in your own
context. Use these delimiters:

THIS IS THE PARENT INVENTORY
[the parent's own declared tool inventory]
END OF PARENT INVENTORY

If the inventory is incomplete or unavailable, state that inside this
block. Do not infer tools from the writer profile.

Make exactly one `task` call:
- agent_type: writer;
- model: the assigned child model;
- name: the test ID followed by "-writer", when supported;
- description: a short description of the writing task;
- prompt: the decoded text of the supplied CHILD_PROMPT JSON string.

The complete decoded CHILD_PROMPT text is the child's prompt argument.
Do not pass a placeholder, a reference, the JSON wrapper, or these parent
instructions. Do not rewrite the path or marker or add instructions.

Wait for the child to finish.

After your single writer task returns, copy its complete returned
response without paraphrasing, shortening, or merging it with your own
statements, between these delimiters:

THIS IS THE WRITER RESPONSE
[the complete response returned by the writer task]
END OF WRITER RESPONSE

Do not replace this content with your own interpretation.
Do not fill in a missing child inventory with your own inventory.
If no writer response was returned, state that explicitly inside the
block. Do not reconstruct one.
If the returned writer response lacks the DECLARED CHILD INVENTORY block,
note that outside the verbatim writer-response block. Do not reconstruct
the inventory, substitute your own inventory, send follow-up instructions,
or retry the child.

The parent's final response must contain both blocks:
- its own inventory block, repeated from its pre-delegation declaration;
- the writer response block.

Do not write, repair, or validate the result file yourself.
Do not launch another child.

## Child prompt template

Replace only {PATH} and {MARKER}.

Before writing, when a progress/response channel is available, report the
exact tool identifiers visible in your own context. Label this as your
declared inventory. If it is incomplete or unavailable, say so. Do not
infer availability from the agent profile.

Write `{MARKER}` to `{PATH}` using an editing tool exposed to you.
The content must be the marker, with or without one final newline.

Do not use terminal commands, shell, scripts, another agent, or any
other writing workaround. Do not create any other file.

Your FINAL response must include both blocks below, even if the inventory
was already reported in a progress message. Progress messages may not be
included in the returned task response.

DECLARED CHILD INVENTORY
[exact tool identifiers visible in your own context, including namespaces
where exposed; source and completeness limitations]
END OF DECLARED CHILD INVENTORY

EDITING RESULT
[editing tool used, if any; whether an editing call was attempted;
the operation and its result, or the precise blockage.
State explicitly when no editing call was attempted.
Do not infer the cause of a blockage.]
END OF EDITING RESULT

If the inventory cannot be obtained, state that limitation inside the
inventory block rather than omitting the block or inferring tools from
the writer profile.

## Verification

Only the assigned child may write its result file.
The orchestrator may create results/ and write report.md.

For this run, inspect the relevant local event records where available
to verify:
- requested and recorded parent/child models;
- selection of writer;
- the actual parent-to-child prompt, especially its path and marker;
- actual child editing calls and their results.

Use actual recorded arguments, not a parent's reconstructed example.
Do not audit previous runs or reconstruct launch history.
Do not dump or compare complete tool schemas.

Read each target file and check its content.
Confirm at the end that writer.agent.md is unchanged.

If evidence is unavailable, identify that specific limitation.
A child inventory block may be present or absent from the returned writer
response. Record that response status separately from any independently
observable progress or runtime evidence, including its source, when
available. Absence from the returned response does not prove that the
child never declared an inventory or had no tools. Do not infer tools from
reported editing tool use or from the absence of an editing call.
A declared inventory is not independently verified runtime evidence.

Separate:
- file result;
- whether the requested test was correctly executed;
- child inventory presence or absence in the returned writer response;
- independently observable progress or runtime evidence, when available;
- limits of available evidence.

A missing runtime inventory does not itself make a test nonconforming.
An inventory block absent from the returned response is a reporting
omission; by itself, it does not establish an execution deviation.
A changed path, marker, model, agent selection, or substantive child
instruction does.

Do not repair or repeat a nonconforming test.

## Report

Write one concise report to report.md containing:

1. Environment and setup:
   available versions, the four model identifiers, writer profile and hash.

2. A matrix with rows and columns in this order:
   gpt-6-luna, gpt-6.1-terra, claude-opus-5.5, claude-fable-5.2.

   Rows are parents; columns are children.
   Mark unselected cells "—".
   For selected cells use:
   CONTENT VALID, FILE ABSENT, CONTENT INVALID, or NOT RUN.
   Append NONCONFORMING only for an actual execution deviation,
   and explain it briefly.

3. One compact evidence row per test:
   - requested models and whether runtime-confirmed;
   - child editing tool and result, or no editing call;
   - file outcome;
   - whether the child inventory block is present in the returned writer
     response or absent from it;
   - independently observable progress or runtime evidence, when
     available, kept separate from returned-response declarations;
   - any execution deviation.

4. Declared inventories:
   preserve the parent and child tool lists for every test.
   Identical lists may be shown once with the applicable test IDs.
   Do not compute parent/child set differences.
   Keep these lists separate from actual recorded tool calls.
   Use the parent inventory block as parent-declared evidence. Extract
   child declarations only from the writer response block. Record whether
   the child inventory block is present in or absent from the returned
   response. If absent, do not describe that as proof the child never
   declared an inventory or had no tools. Record independently observable
   progress or runtime evidence separately, when available; do not use it
   to reconstruct a returned-response declaration. Do not infer inventory
   contents from the parent's tools or the writer profile. Treat both
   parent and child declarations as declarations, not independently
   verified runtime inventories.
   Do not reproduce the complete writer response in report.md unless
   needed; preserve the requested inventory lists and concise result
   summary.

5. A short conclusion describing the observed pattern, without assuming
   that heterogeneous delegation is the cause.
   Include trace location and parent/child call IDs where available,
   without reproducing the full logs.

Preserve the result files. Do not create additional reports or run
additional combinations.

After saving report.md, immediately return the matrix and a short
summary of any execution deviations.