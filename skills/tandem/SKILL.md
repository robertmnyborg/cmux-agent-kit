---
name: tandem
description: Launch the tandem split workspace (broad-context agent left, claude right) by running cmux-tandem. Trigger when the user's entire prompt is "tandem" or "/tandem", or says "open tandem" / "launch tandem" / "start tandem". Do not trigger on prompts that merely mention tandem.
allowed-tools: Bash(cmux-tandem*)
---

Run `cmux-tandem` with the Bash tool. Report the workspace id from its stdout on one line. Do nothing else and ask no questions.
