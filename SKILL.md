---
name: implement-tickets
description: "Orchestrate a spec's tickets: one implement subagent per ticket, then a global two-axis review."
disable-model-invocation: true
---

# Implement tickets

Orchestrate the tickets derived from one spec. You are the orchestrator: you dispatch, verify, and review — you write no implementation code yourself. Work tickets **one at a time**, blockers first, so commits stay linear.

Use the `subagent` tool for every dispatch. A subagent shares none of your context: its prompt must be self-contained.

## 1. Survey

Read the spec the user names. List its tickets from the issue tracker (local `.scratch/<feature>/issues/`, or the remote tracker), ordered blockers-first.

Done when: you hold the ordered ticket list, and it is written in this conversation as a ledger you maintain — one line per ticket: `pending | dispatched(<agent-name>) | done(<commit>)`.

## 2. Dispatch the next ticket

Take the first pending ticket whose blockers are all done. Dispatch one subagent for it. The prompt carries:

- The ticket reference, the spec's location, the working directory, and the branch.
- The implement contract, quoted in full. Preface it with: "The following is the complete text of the `implement` skill. It is all you need — there is no skill to load and no further document to find. Follow it exactly:"

  > Implement the work described by the user in the spec or tickets.
  >
  > If the user passes a ticket reference, fetch it from the issue tracker and state its title before starting. If the reference is ambiguous, ask.
  >
  > Call the Skill tool with "tdd" where possible, at pre-agreed seams.
  >
  > Run typechecking regularly, single test files regularly, and the full test suite once at the end.
  >
  > Once done, call the Skill tool with "code-review" to review the work.
  >
  > Commit your work to the current branch.

- A report-back requirement: its final message states the commit hash, what it built, and every deviation from the ticket.

Then **end your turn**. The subagent's completion notice starts your next turn — that notice, not polling, is the loop.

Done when: the subagent is dispatched and the ledger shows `dispatched(<name>)`.

## 3. On each completion notice

Verify before believing the report:

- The reported commit exists on the branch (`git log`).
- The diff touches the ticket's scope and nothing far outside it.

If the subagent reports a failure or blocker, steer it with `send_message`; if the ticket itself turns out to be wrong, stop and ask the user. If verification passes, mark `done(<commit>)` and return to step 2. If no tickets remain, go to step 4.

Done when: every ticket is `done(<commit>)` in the ledger.

## 4. Global review

Per-ticket review checks a diff against one ticket; the gaps *between* tickets only show against the whole spec. Dispatch two reviewers in parallel over the cumulative diff (base of the first ticket..HEAD), each given the spec:

- **standards-reviewer** — the Standards axis of `code-review`: the repo's coding standards applied to the whole diff.
- **spec-reviewer** — the Spec axis: every requirement in the spec is met somewhere in the build; report each gap or deviation with file evidence.

Each reviewer returns a findings list: what is wrong, where, and what the spec or standard demands. An empty list is a valid result.

Then end your turn; the two completion notices resume you.

Done when: both reviewers have reported.

## 5. Fix deviations

For each confirmed finding, dispatch a fix subagent scoped to that finding: the evidence, the spec passage it violates, and the expected behaviour. It commits its fix to the same branch. Independent findings may dispatch in parallel.

After fixes land, re-run the reviewer whose axis the finding belonged to on the new diff.

Done when: both axes report no findings. Report the close-out: tickets built with commits, review outcome, fixes applied.
