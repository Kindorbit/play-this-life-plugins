---
name: play-this-life
description: Use when the user talks about their goals, habits, chores, routines, workouts or today's plan, or wants to add, change or record tasks in Play This Life.
---

Play This Life keeps a person's goals, habits, chores, routines and workouts as tasks with schedules
and recorded progress. The connected server exposes the account the user signed in with.

How to work with it:

1. Call `read_task_context` first. It returns local date, time and timezone, the next 15 days,
   the account's task contexts and recent tasks. Dates are local `YYYY-MM-DD`; times are epoch
   milliseconds.
2. Find or read before you change. `find_tasks` searches titles; `read_tasks` returns hierarchy,
   schedules, progress and the calendar. Pass each task's `updatedAt` as `expectedUpdatedAt` on
   edits, progress and lifecycle changes.
3. Create with `create_tasks`. A simple task needs only a title; add a schedule for habits and
   chores, children for goals with steps. Use `preview_tasks` for a complex schedule before saving.
4. Record what the user finished with `record_task_progress`; undo a mistaken record with
   `undo_task_completion`. Archive, trash or restore with `change_task_lifecycle`; Trash is
   recoverable.
5. Answer "what did I get done" with `read_wins` and "what slipped" with `read_work_history`.

Never invent completions or results the user did not report. If a write returns that the account
cannot edit right now, say so and suggest opening the Play This Life app.
