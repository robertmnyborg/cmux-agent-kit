---
name: spinoff
description: Spin a side thread off the current session into a fresh Claude tab in the same cmux pane, with a handoff brief, and record the parent->child link for the Agent Overworld map. Use when the user says "/spinoff", "spin this off", "open a new session for this", "deep dive this in a fresh session", or wants to continue a thread without bloating the current context.
---

# Spinoff

Opens a child session next to this one. The child starts from a written brief, not from this transcript.

## Procedure

1. Name the thread in 2-4 words. Use the user's words when they gave a name.
2. Write the brief to `~/.overworld/handoffs/<YYYY-MM-DD>-<kebab-title>.md`.
3. Put these sections in the brief: Goal, What is already decided, Facts and numbers with their sources, Files and paths, Open questions, First step.
4. Keep the brief under 60 lines. Include only what the child needs for this thread.
5. Run `cmux-spinoff "<Title>" <brief path>` from this session's working directory.
6. Add `--focus` as the third argument only when the user asks to jump to the new tab.
7. Report the tab title and the brief path in one line.

## Rules

- Do not continue the side thread in this session after the spinoff.
- Do not copy secrets or credentials into the brief. Name the file or env var that holds them.
- If `$CMUX_SURFACE_ID` is empty, the session is not in cmux. Tell the user and stop.
