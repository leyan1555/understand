# understand

Make AI think before it acts. This skill forces AI to clarify its goal, align with you, and show its full plan and process before execution. No more black-box surprises.

`understand` is a gate for coding agents. When a request contains the token `/understand`, the agent reports the task goal, the numbered process, and every material ambiguity, then waits. The task starts on a later message that accepts that report.

## What it does

- States the ask and the observable end state.
- Numbers each step, then names what will change and what will stay untouched.
- Turns a vague prompt into an adopted reading you can correct before any work starts.

A request such as "help me troubleshoot the login loop and make sure it doesn't happen again" must say which symptom is in scope, what "doesn't happen again" means, and which reading the agent will follow.

## Install

```bash
npx skills add leyan1555/understand
```

The skill is the root `SKILL.md` in this repository. To install it by hand, copy that file into your agent's skills directory, for example `~/.agents/skills/understand/SKILL.md`.

## Usage

Put `/understand` anywhere in the request:

```text
Add a logout button to the settings page /understand
```

The agent answers with the report and stops. The button is added only after you accept the plan.

The bare word "understand" does not trigger the skill. `/understand`, `/Understand`, and `/UNDERSTAND` do.

## The report

The report uses your language and always has these three sections:

1. **Task goal.** What you asked for, and what is true when the task is done.
2. **Process detail.** Steps `1.` `2.` `3.`, plus the files and systems that will change and the ones that will stay untouched.
3. **Task expansion.** Each ambiguity, why it changes the work, and the reading the agent will use if you approve the report as written.

## Confirmation

Work starts only after a later message that accepts the report and adds no new constraint. These count:

- `我觉得没问题了，你可以开始了`
- `可以开始了`
- `开始吧`
- `approved, start`
- `go ahead`
- a short reply such as `好`, `可以`, or `ok`

A reply that changes the scope is a revision, even if it also says to start. `可以开始，但别动测试` sends the agent back to an updated report. The revised plan starts on the next confirmation.

A "you can start" in the same message as `/understand` does not count. You have not seen the report yet.
