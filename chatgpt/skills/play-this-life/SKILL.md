---
name: play-this-life
description: Use when the user talks about their goals, habits, chores, routines, workouts or today's plan, or wants to add, change or record tasks in Play This Life.
---

Play This Life keeps a person's goals, habits, chores, routines and workouts as tasks with schedules and recorded progress. Tools act only on the account connected through OAuth.

## Read, act and verify

Call read_task_context first for trusted local date, time and timezone. Its localDate is the person's day, which starts at dayStart (for example 03:00), so at 01:00 it is still the previous day; localDateTime is the wall clock. It is a summary, not the full account. Use find_tasks to select existing tasks and read_tasks for their current definitions, schedules and occurrences. Follow nextCursor when a search is incomplete. A title is not a unique identity.

A clear user request authorizes the necessary changes within that request; respect the client's tool approval controls. Ask one concise question when plausible interpretations would save materially different results. Never invent a clock time, target, completion or actual measurement. Explain unsupported requests without substituting a different saved result.

Before changing an existing task, read its latest definition.updatedAt and pass it as expectedUpdatedAt. On a conflict, read again and reassess. Verify returned saved facts before reporting success. Creation returns saved settings and schedules without calculating calendar occurrences; use read_tasks with explicit dates only when calendar verification is needed. Use IDs from the saved create result and plannedTarget from a dated saved read.

On validation failure, correct the documented input while preserving the user's requested result. Never drop a requested schedule, recurrence, deadline or target to make a write pass. If it cannot be represented, stop and explain what failed. After an uncertain write or lost response, read to establish whether it saved before retrying; each write call is a new operation and can duplicate work. Report successful, failed and pending changes separately.

For dated calendar reads, pass from/to together (exclusive to, at most 93 days) without scope, taskTypes or viewTypes. Optional taskIds selects saved tasks within that range. Collection reads are a separate input mode: scope: current requires taskTypes or viewTypes; scope: projects selects the project collection. Neither collection mode accepts from/to, taskIds or includeArchived. Never combine those modes. completionHistory is explicitly paged; use saved task IDs and follow each history cursor.

Today read example (illustrative dates; replace both from trusted local context): {"from":"2026-10-27","to":"2026-10-28"}

## Task purpose and hierarchy

taskType describes purpose, independently of recurrence: task = specific work; chore = upkeep; habit = a behavior to establish; routine = steps performed together; event = a dated occurrence; goal = a desired outcome; project = a finite effort; list = ongoing organization. Goals cannot repeat. Repeating a task does not by itself make it a habit. A duration in the title does not automatically create a measured target.

create_tasks takes tasks and optional activities. A simple task needs only title. For a hierarchy, give parents a local ref and create them before children; a child names its parent with parentRef: "parent-ref" or parentTaskId: "saved-id". Nesting never completes a parent. For a repeating group, put the repeat on the parent with childSurfacing: "inside_parent"; its new unscheduled one-off steps then join its calendar/quota occurrences automatically. Only one task along an ancestor-descendant path owns a repeat.

## Timing and recurrence inputs

Timing is schedule: {sch_timezone, sch_scheduled, sch_deadline}; a repeat adds recurrence: {rule} and still needs schedule for its timezone. sch_timezone is a trusted IANA zone. sch_scheduled and sch_deadline are both required whenever schedule is supplied: local YYYY-MM-DD or YYYY-MM-DDTHH:mm, or null if absent. A deadline is independent of an intended time; never infer one from the other. Timestamps such as completedAt and updatedAt are epoch milliseconds. Each saved time a user may need also has a sibling <field>Local (YYYY-MM-DDTHH:mm in the nearest enclosing timezone): report those and never convert epoch values yourself. An all-day value is its YYYY-MM-DD date, not a clock time. lengthSeconds is how long a task is planned to take; on a timed task it is its calendar block, not a target. A time before the day start belongs to the previous day: with a 03:00 day start, Monday 01:00 is Sunday night.

Every repeat rule needs sch_kind, sch_active_from (local YYYY-MM-DD) and sch_miss_policy (skip, carry_one or backlog). Omit sch_active_until unless the user supplies an end date; backlog requires explicit intent and an optional sch_backlog_limit from 1 to 30.

- calendar: fixed dates/weekdays/times; also supply sch_rrule, sch_all_day and, unless all-day, sch_time (local HH:mm). The first rule date on or after sch_active_from starts the repeat. Supported RRULE keys include FREQ=DAILY|WEEKLY|MONTHLY, INTERVAL, WKST, BYDAY, BYMONTHDAY and BYSETPOS; use sch_active_until instead of COUNT or UNTIL. For weekday repeats use FREQ=WEEKLY;BYDAY=MO,TU,WE,TH,FR. Without an explicit clock time, use an all-day repeat and preserve wording such as morning in title/description.
- relative: measured from completion; supply sch_delay_days (1 to 3650) and optionally sch_relative_trigger: attempt_completion or fulfillment. A measured relative requirement uses fulfillment.
- period: a quantity within each calendar day/week/month, without selected occurrence days; supply sch_period_unit: day, week or month.
- none: independent repeated acts without a calendar quota.

A repeat's evaluation goes inside recurrence.rule; do not also supply task-level evaluation. eva_kind: manual records explicit completion; count also needs eva_target; sum needs eva_target and an owned eva_measurement_id. Nonrepeating tasks may use task-level evaluation. Never infer reported results from these planned requirements.

Example create_tasks input for an explicitly requested weekday 09:00 habit. These dates and timezone are illustrative; replace them using the user's request and trusted account context:

```json
{"tasks":[{"title":"Stretch 10 minutes","taskType":"habit","schedule":{"sch_timezone":"Asia/Kolkata","sch_scheduled":"2026-10-27T09:00","sch_deadline":null},"recurrence":{"rule":{"sch_kind":"calendar","sch_active_from":"2026-10-27","sch_miss_policy":"skip","sch_all_day":false,"sch_rrule":"FREQ=WEEKLY;BYDAY=MO,TU,WE,TH,FR","sch_time":"09:00"}}}]}
```

Check the returned saved task IDs and schedules. To check complex timing, call read_tasks with those taskIds and from/to; confirm the generated dates, start, timezone and requested rest days, and correct mistakes with edit_tasks.

## Editing saved tasks

edit_tasks takes tasks: [{taskId, expectedUpdatedAt, changes}], plus optional range. Omitted changes stay unchanged; null clears nullable values. Combine changes to the same task in one entry. changes take create's fields except refs, parent, beforeTaskId and passes, plus placement: {parentTaskId, beforeTaskId}, exceptions and skillIds. Repeating changes use changes.recurrence: {rule, scope, effectiveDate?, timezone?, resetContinuity?}. scope is this_and_future, all_uncompleted or from_date; only from_date takes effectiveDate. Use the saved rule and user intent to choose scope, preserving recorded work and unrelated fields. A repeat's timezone changes with its recurrence. Verify returned effective dates and pending changes before describing a rule as active.

An occurrence-only skip/reschedule uses changes.exceptions: [{action: "skip"|"reschedule"|"clear", plannedTarget, scheduled?}]. Copy the intended occurrence's plannedTarget from a saved read. Do not replace a whole recurring series to change one date. Preserve started or recorded work. For detailed active-work or workout-result operations absent from this connector, direct the user to Play This Life rather than inventing tool names or actuals.

## Recording and undoing work

For reported work, find/read the saved task and select the intended local date. record_task_progress requires taskId and expectedUpdatedAt. For scheduled work copy that occurrence's plannedTarget exactly, either {windowId} or {occurrenceId}, from a saved result/read. Omission or null records independent unplanned work and must not substitute for a requested scheduled completion. completedAt is the actual epoch-millisecond completion time; omit for now. quantity is the reported completion quantity, not a duration or other measurement.

measurements uses [{measurementId, actualValue}] with owned IDs and storage units from reads; supply only actual results reported by the user. Planned targets are not actual results. A sum target requires its matching measurement. If an operation needs unsupported detailed active-work tools, explain that limitation and use the app. Never invent completion to satisfy a target.

Keep the exact completionId returned by the write. To undo, use undo_task_completion with that completionId, the taskId and its latest expectedUpdatedAt; task IDs and window IDs are not completion IDs. Verify the saved response. Rewards may still be pending; report awarded XP only from returned evidence.

## Reviews and lifecycle

read_wins answers what was recorded and awarded in a local day/week/month/all period; read_work_history answers met, late, missed, skipped, open or interrupted expectations. Use their returned timezone and coverage. Missing historical coverage is unknown, not zero. Follow pagination without adding whole-period summaries repeatedly. Pending assessments are not awarded XP.

change_task_lifecycle with taskId, expectedUpdatedAt and action: archive, trash or restore also affects applicable descendants and keeps history. Trash is recoverable; this connector cannot permanently purge tasks. Only the connected account is accessible. If the account cannot edit right now, explain the returned limitation and suggest opening Play This Life.
