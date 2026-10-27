---
name: play-this-life
description: Use when the user talks about their goals, habits, chores, routines, workouts or today's plan, or wants to add, change or record tasks in Play This Life.
---

Play This Life keeps a person's goals, habits, chores, routines and workouts as tasks with schedules and recorded progress. Tools act only on the account connected through OAuth.

## Read, act and verify

Call read_task_context first for trusted local date, time, timezone and task contexts. It is a summary, not the full account. Use find_tasks to select existing tasks and read_tasks for their current definitions, schedules and occurrences. Follow nextCursor when a search is incomplete. A title is not a unique identity.

A clear user request authorizes the necessary changes within that request; respect the client's tool approval controls. Ask one concise question when plausible interpretations would save materially different results. Never invent a clock time, target, completion or actual measurement. Explain unsupported requests without substituting a different saved result.

Before changing an existing task, read its latest definition.updatedAt and pass it as expectedUpdatedAt. On a conflict, read again and reassess. Verify returned saved facts and calculated dates before reporting success. A preview is unsaved: never use its temporary task IDs or occurrence selectors for a write. Use IDs and plannedTarget from the saved create result or a subsequent read.

On validation failure, correct the documented input while preserving the user's requested result. Never drop a requested schedule, recurrence, deadline or target to make a write pass. If it cannot be represented, stop and explain what failed. After an uncertain write or lost response, read to establish whether it saved before retrying; each write call is a new operation and can duplicate work. Report successful, failed and pending changes separately.

## Task purpose and hierarchy

taskType describes purpose, independently of recurrence: task = specific work; chore = upkeep; habit = a behavior to establish; routine = steps performed together; event = a dated occurrence; goal = a desired outcome; project = a finite effort; list = ongoing organization. Goals cannot repeat. Repeating a task does not by itself make it a habit. A duration in the title does not automatically create a measured target.

create_tasks takes tasks and optional activities. A simple task needs only title. For a hierarchy, give parents a local ref and create them before children; a child's parent is {kind: "plan", ref: "parent-ref"} or {kind: "existing", taskId: "saved-id"}. Nesting alone does not inherit timing or complete a parent. For a repeating group, put the repeat on the parent, childSurfacing: "inside_parent" on it, and surfacing: "inside_parent" on its unscheduled steps. Only one task along an ancestor-descendant path owns a repeat. Reuse taskContext IDs from the account catalog.

## Timing and recurrence inputs

Creation uses scheduleIntent: {sch_timezone, sch_scheduled, sch_deadline, sch_rule?}. sch_timezone is a trusted IANA zone. sch_scheduled and sch_deadline are both required whenever scheduleIntent is supplied: local YYYY-MM-DD or YYYY-MM-DDTHH:mm, or null if absent. A deadline is independent of an intended time; never infer one from the other. Other timestamp fields such as completedAt and updatedAt use epoch milliseconds.

The repeat field is sch_rule, not sch_recurrence. Every repeat rule needs sch_kind, sch_active_from (local YYYY-MM-DD) and sch_miss_policy (skip, carry_one or backlog). Omit sch_active_until unless the user supplies an end date; backlog requires explicit intent and an optional sch_backlog_limit from 1 to 30.

- calendar: fixed dates/weekdays/times; also supply sch_rrule, sch_dtstart_local (YYYY-MM-DDTHH:mm:ss) and sch_all_day. Supported RRULE keys include FREQ=DAILY|WEEKLY|MONTHLY, INTERVAL, WKST, BYDAY, BYMONTHDAY and BYSETPOS; use sch_active_until instead of COUNT or UNTIL. DTSTART must match the rule. For weekday repeats use FREQ=WEEKLY;BYDAY=MO,TU,WE,TH,FR. All-day repeats use sch_all_day: true and a T00:00:00 DTSTART. Without an explicit clock time, use an all-day repeat and preserve wording such as morning in title/description.
- relative: measured from completion; supply sch_delay_days (1 to 3650) and optionally sch_relative_trigger: attempt_completion or fulfillment. A measured relative requirement uses fulfillment.
- period: a quantity within each calendar day/week/month, without selected occurrence days; supply sch_period_unit: day, week or month.
- none: independent repeated acts without a calendar quota.

A repeat's evaluation goes inside sch_rule; do not also supply task-level evaluation. eva_kind: manual records explicit completion; count also needs eva_target; sum needs eva_target and an owned eva_measurement_id. Nonrepeating tasks may use task-level evaluation. Never infer reported results from these planned requirements.

Preview complex creation with preview_tasks: {proposal: {operation: "create", input: {tasks, activities?}, range: {from, to}}}. For edits use operation: "update" and the edit_tasks input without batchId or range. to is exclusive. Inspect generated dates, start, timezone and requested rest days, then pass the same tasks/activities to create_tasks. Previews create no domain rows; calendarContext.complete: false means overlaps are not fully checked. Narrow an oversized preview rather than silently omitting requested work.

Example preview for an explicitly requested weekday 09:00 habit. These dates and timezone are illustrative; replace them using the user's request and trusted account context:

```json
{"proposal":{"operation":"create","input":{"tasks":[{"title":"Stretch 10 minutes","taskType":"habit","scheduleIntent":{"sch_timezone":"Asia/Kolkata","sch_scheduled":"2026-10-27T09:00","sch_deadline":null,"sch_rule":{"sch_kind":"calendar","sch_active_from":"2026-10-27","sch_miss_policy":"skip","sch_all_day":false,"sch_rrule":"FREQ=WEEKLY;BYDAY=MO,TU,WE,TH,FR","sch_dtstart_local":"2026-10-27T09:00:00"}}}]},"range":{"from":"2026-10-27","to":"2026-11-03"}}}
```

After this preview, create_tasks receives proposal.input plus optional range, not the proposal wrapper. Check the returned saved task IDs and occurrences before claiming the habit is scheduled.

## Editing saved tasks

edit_tasks takes tasks: [{taskId, expectedUpdatedAt, changes}], plus optional range. Omitted changes stay unchanged; null clears nullable values. Combine changes to the same task in one entry. changes.schedule is {sch_timezone, sch_scheduled, sch_deadline}; repeating changes instead use changes.recurrence: {rule, scope, effectiveDate?, timezone?, resetContinuity?}. rule is the full sch_rule structure above. scope is this_and_future, all_uncompleted or from_date; only from_date takes effectiveDate. Use the saved rule and user intent to choose scope, preserving recorded work and unrelated fields. A repeat's timezone changes with its recurrence. Verify returned effective dates and pending changes before describing a rule as active.

An occurrence-only skip/reschedule uses changes.exceptions: [{action: "skip"|"reschedule"|"clear", plannedTarget, scheduled?}]. Copy the intended occurrence's plannedTarget from a saved read. Do not replace a whole recurring series to change one date. Preserve started or recorded work. For detailed active-work or workout-result operations absent from this connector, direct the user to Play This Life rather than inventing tool names or actuals.

## Recording and undoing work

For reported work, find/read the saved task and select the intended local date. record_task_progress requires taskId and expectedUpdatedAt. For scheduled work copy that occurrence's plannedTarget exactly, either {windowId} or {occurrenceId}, from a saved result/read. Omission or null records independent unplanned work and must not substitute for a requested scheduled completion. completedAt is the actual epoch-millisecond completion time; omit for now. quantity is the reported completion quantity, not a duration or other measurement.

measurements uses [{measurementId, actualValue}] with owned IDs and storage units from reads; supply only actual results reported by the user. Planned targets are not actual results. A sum target requires its matching measurement. If an operation needs unsupported detailed active-work tools, explain that limitation and use the app. Never invent completion to satisfy a target.

Keep the exact completionId returned by the write. To undo, use undo_task_completion with that completionId, the taskId and its latest expectedUpdatedAt; task IDs and window IDs are not completion IDs. Verify the saved response. Rewards may still be pending; report awarded XP only from returned evidence.

## Reviews and lifecycle

read_wins answers what was recorded and awarded in a local day/week/month/all period; read_work_history answers met, late, missed, skipped, open or interrupted expectations. Use their returned timezone and coverage. Missing historical coverage is unknown, not zero. Follow pagination without adding whole-period summaries repeatedly. Pending assessments are not awarded XP.

change_task_lifecycle with taskId, expectedUpdatedAt and action: archive, trash or restore also affects applicable descendants and keeps history. Trash is recoverable; this connector cannot permanently purge tasks. Only the connected account is accessible. If the account cannot edit right now, explain the returned limitation and suggest opening Play This Life.
