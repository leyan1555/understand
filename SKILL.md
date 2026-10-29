---
name: understand
description: >-
  Gate any user request that contains the token /understand. Before other
  work, report the task goal, a numbered process that names what will change
  and what will stay untouched, and a task expansion that specifies each
  ambiguity and the adopted reading, then wait for a later start confirmation
  such as "我觉得没问题了，你可以开始了". Use when the user writes
  /understand. The bare word understand does not trigger this skill.
---

# Understand

`/understand` opens a **gate**. The gated task is everything else in that user message. The understand turn delivers one report and ends. The gated task starts on a later turn, only after a confirmation that accepts this report.

A report followed by implementation in the same turn is a failed gate. Sending the report is the whole turn.

## When this skill loads

Load and follow this skill in full before any other action when the user message contains the token `/understand`.

Match the token with a leading slash, case-insensitive: `/understand`, `/Understand`, `/UNDERSTAND`. It may sit at the start, middle, or end of the message. The surrounding words are the gated task, not part of the token.

These do not open the gate:

- the bare word "understand", "understanding", or "make sure you understand"
- a request to be careful, to restate, or to ask questions, unless the slash token is also present
- an earlier `/understand` whose gate is already closed

If the message is only the token, the gated task is the unfinished task already in the conversation. If there is no unfinished task, the report says that and asks what to gate. That question is the end of the turn.

## The understand turn

Do these in order. The completion criterion of each step is on the same line.

1. Mark the gated task and strip the token. Done when you can point at the exact request being gated.
2. Do read-only reconnaissance when the task depends on the repo, files, or a live system. Done when the report can name real paths and a real non-touch list, or when the task has no environment to inspect.
3. Write the report in the user's language, using the contract below. Done when all three sections are present, the process is numbered, every material ambiguity has an adopted reading, and the touch list is concrete.
4. Send the report and end the turn. Done when the user has the report and no mutating action has run.

The user-facing message is the report. Start with one sentence that the token was found and this turn is the report only. End with one sentence that work starts only after they confirm. Do not add a second reply, a partial implementation, or "while you read this I will...".

### Read-only reconnaissance

Inspect before reporting whenever naming files from memory would be a guess. Allowed: read file contents, search, list directories, read git status / diff / log, read docs, and look up facts needed to state the plan.

The understand turn does not mutate. That includes edits, new files, deletes, installs, migrations, commits, pushes, deploys, sent messages, created issues or resources, and any subagent or shell whose job is to change something. A read-only look is part of the report. A write is part of the gated task and waits.

If a full investigation is itself the task, learn enough to name the plan, the likely surfaces, and the unknowns. Leave the rest of the investigation on the post-confirmation side of the gate.

## The report

Write the report in the language of the gated task. When the task is Chinese, use these headings verbatim:

1. 任务目标
2. 过程细节
3. 任务拓展

When the task is English, use:

1. Task goal
2. Process detail
3. Task expansion

For any other language, translate those three labels and keep the numbers and order. Always these three sections, in this order, even when a section is short.

### 1. Task goal

State both:

- **Ask.** What the user told the agent to do, in their terms. Separate a direct restatement from your interpretation, and mark the interpretation.
- **Done when.** The ideal end state as observable facts: what exists, what changed, what a reviewer can check. A feeling ("it works better") is not a done-when. Name the artifact, behavior, or answer that will exist.

If they named a success condition, use it. If they did not, propose one and repeat it under task expansion as an adopted reading.

### 2. Process detail

This section is the whole method, not a sketch. Use a numbered list, `1.` `2.` `3.`, one action per step, in execution order. Each step says what happens and how you will know that step is done. Include the check you will run after the last step.

After the numbered list, add two labeled lists:

- **Will change.** Concrete files, commands, services, records, or messages. "The codebase" is too vague when you can name paths. If the task creates something new, name the path you will create.
- **Will leave untouched.** The nearby things a reasonable agent might also edit: other features, configs, dependencies, git state, unrelated files, and anything the user did not ask to alter. Name them.

If a step is conditional, write the condition in that step. Hidden branches belong in the report, not in the later implementation.

### 3. Task expansion

Specify and clarify every material ambiguity, then lock an **adopted reading**: the interpretation you will execute if they confirm this report with no edits.

An ambiguity is material when two reasonable readings would change the files touched, the done-when, the risk, or the size of the task. Vague prompts are this section's main job. Treat each of these as unresolved until you specify them:

- missing object ("fix the bug", "troubleshoot xxx")
- missing done-when ("make sure xxx doesn't happen again", "clean this up", "make it production-ready")
- missing scope (which app, environment, page, user, or file)
- a word that could mean several controls: a code fix, a test, a monitor, a revert, a rewrite, or a process change
- two tasks joined by "and" that can be separated
- an example that might be the whole scope or only an illustration

For each material ambiguity, use this shape:

- **Ambiguity.** The phrase and the two or more readings.
- **Why it matters.** What would diverge in the work.
- **Adopted reading.** The reading you will use on confirmation. Pick the smallest reading that fully satisfies the ask. State what you are leaving in the other readings.

Also list **residual assumptions**: environment, versions, "change this, not that", and defaults you filled in. Assumptions are adopted readings with lower stakes. They still belong here so the user can correct them before work starts.

When the prompt is already specific, write "No material ambiguity." and still list residual assumptions. Do not invent fake ambiguity to look thorough. Do not resolve ambiguity by starting the work.

Canonical vague shape: "help me troubleshoot X and make sure X doesn't happen again."

Specify at least:

- what X is, and which symptom you will treat as X
- that the cause is currently unknown, plus the surfaces you will inspect first
- the reading of "doesn't happen again": a regression test, a guard in code, a monitor, a doc, or more than one of those
- how you will show the cause is actually fixed

## Confirmation

After the report, wait. Only a **later** user message can close the gate. A confirmation inside the same message as `/understand` does not count: they have not seen the report yet. A confirmation for an older task does not carry forward.

### What counts

A confirmation clearly accepts this report and authorizes the gated task. These count when they refer to the report just sent and add no new constraint:

- `我觉得没问题了，你可以开始了`
- `可以开始了`
- `开始吧`
- `你理解的没问题，可以开始了`
- `approved, start`
- `go ahead`
- `LGTM, start`
- a short direct reply with no new question and no new constraint: `好`, `好的`, `可以`, `ok`, `okay`, `approved`

Also a clear authorization to drop the wait and execute the agreed plan, such as `算了直接做` or `just do it`, still with no new constraint.

Restating the agreed task counts. Adding a detail that was already in the report counts.

### What stays shut

The gate stays shut, and the gated task stays unstarted, when the next message is:

- silence, `嗯`, `hmm`, or an emoji with no words
- a question about the report
- `continue` or `go on` when it can mean "keep explaining" rather than "start"
- approval of only one section
- a new unrelated request

Answer a question about the report, then keep waiting. For an unrelated new request, leave the gated task unstarted and handle the new request on its own. If that new request is the gated task again without a confirmation, send the waiting status and keep the gate shut. Ask one direct question when a short reply is genuinely ambiguous: whether they want you to start the agreed plan.

### Revisions

A reply that adds, removes, or changes a constraint is a revision, even when it also says to start. Examples: `可以开始，但别动测试`, `go ahead but skip the migration`, `好，另外把日志也加上`.

On a revision: update the ask, the done-when, the numbered steps, the two touch lists, and the adopted readings. Send the revised report. Wait again. The revised plan starts only after a later confirmation that adds nothing new.

## After confirmation

Execute the agreed plan. The report is the spec.

Stay inside **Will change**. A discovery that the plan is wrong, unsafe, or incomplete reopens the gate: stop, report the delta (what you found, what you now propose to change, what still stays untouched), and wait. Do not silently widen the touch list.

When the agreed check passes, tell the user what changed and how you checked it. That closes the task.

## Special cases

**Several tasks in one message.** One report. Number the tasks inside the task goal and keep one process list that shows the order. One confirmation covers every task in that report. If they confirm only some tasks, do only those, and restate the ones left waiting.

**Mid-task token.** If they send `/understand` while work is already in progress, stop that work immediately. The report covers what is already done, what remains, what is untouched, and the adopted reading for the remainder. Leave finished work in place. Revert only if they ask. Wait for a new confirmation before the next change.

**Disallowed work.** The gate does not override a higher-priority safety rule. Report the ask, state that the work will not be done, and give the high-level reason. Omit procedures, parameters, and partial steps. Confirmation does not start that work.

**Pure questions.** The answer is the gated task. The report plans the answer: the scope, the sources, and what you will leave out. The answer itself waits for confirmation.

## Failed gate

Each of these is a failed gate. If you are about to do one, send or correct the report instead.

- editing, installing, committing, or otherwise mutating during the understand turn
- answering the gated task in the same turn as the report
- a process section with no numbers, or a touch list that says only "relevant files"
- a vague prompt with no adopted reading
- treating `嗯`, silence, or a revision-plus-start as confirmation
- treating a same-message "you can start" as confirmation
- continuing because the plan feels obvious or the edit is small

Smallness is not an exemption. A one-line change still waits.

## Examples

Imitate the shape. Swap in the real task. Keep the user's language.

### Vague repair

User: `help me troubleshoot the login redirect loop and make sure it doesn't happen again /understand`

Report:

```markdown
This message contains /understand, so this turn is the report only.

## 1. Task goal

**Ask.** Find why login enters a redirect loop, fix that cause, and make a repeat of this loop detectable before it ships again.

**Done when.** Login from a logged-out session reaches the app without a redirect cycle. A failing test or an equivalent guard exists for the loop you found. The report of the cause names the request cycle and the file that broke it.

## 2. Process detail

1. Reproduce the loop and record the redirect chain. Done when the cycle is written down request by request.
2. Trace that chain to the code that issues the bad redirect. Done when one cause is identified, or the remaining candidates are listed.
3. Change only the code on that cause. Done when the reproduced steps no longer loop.
4. Add a regression test for that chain. Done when the test fails on the old behavior and passes on the fix.
5. Run the login tests. Done when they pass, or the failures are reported with no further edits.

**Will change.** The redirect/auth file identified in step 2. One new or existing test file for that chain.

**Will leave untouched.** Session storage, the login UI copy, unrelated routes, dependencies, and git history.

## 3. Task expansion

**Ambiguity.** "Troubleshoot the login redirect loop" does not name the environment, the role, or the URL.

**Why it matters.** A local infinite redirect and a production OAuth loop are different fixes.

**Adopted reading.** The loop in this repo's local login flow. If reproduction shows it only happens in a deployed environment, stop and re-report before changing code.

**Ambiguity.** "Make sure it doesn't happen again" can mean a test, a rewrite of auth, a monitor, or a runbook.

**Why it matters.** Those touch different files and different definitions of done.

**Adopted reading.** A regression test for the chain you found. No monitor, no auth rewrite, no new docs.

**Residual assumptions.** The app runs locally with the repo's normal test command. No production credentials are required.

I will start only after you confirm this plan.
```

### Specific edit

User: `In Header.tsx, rename the button label Save to Publish. /understand`

```markdown
This message contains /understand, so this turn is the report only.

## 1. Task goal

**Ask.** Change the Header button label from Save to Publish.

**Done when.** Header.tsx renders Publish where it previously rendered Save, and no other label changed.

## 2. Process detail

1. Find the Save label in Header.tsx. Done when the exact string and its element are known.
2. Replace that label with Publish. Done when the source shows Publish.
3. Check for a test or story that asserts the old string. Done when those assertions match, or none exist.

**Will change.** Header.tsx. A test or story only if it asserts the old label.

**Will leave untouched.** The save handler, the API, other buttons, and copy outside Header.

## 3. Task expansion

No material ambiguity.

**Residual assumptions.** "Save" is the visible English label, not an i18n key. If it is a key, stop and re-report before editing.

I will start only after you confirm this plan.
```

### Revision, then confirmation

User after a report: `可以开始，但别动测试`

That is a revision. Re-report with tests moved to **Will leave untouched**, drop the test step, and wait. The following `可以开始了` is the confirmation that starts the revised plan.

User after a report: `我觉得没问题了，你可以开始了`

That is a confirmation. Start the plan that was just reported. Do not re-report first.
