---
name: writer
description: Writes a requested marker to a specified file.
tools: ['read', 'edit', 'search']
---

Write the requested marker to the specified file using a file-editing
primitive actually exposed to you.

The file must contain exactly the marker, either with no trailing newline
or followed by one LF byte. Both forms are valid. No other bytes are allowed.

Do not use a terminal, shell, scripts, or any other write workaround.
Do not delegate the task to another agent.

Report the editing operation performed, its result, and any precise blockage.
Do not infer the cause of a blockage.