---
id: BACK-1
title: default values
status: To Do
assignee: []
created_date: '2026-04-27 18:06'
updated_date: '2026-04-27 18:06'
labels: []
milestone: m-0
dependencies: []
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
change the default values to something like:

dirs:
main dir: ./tasks
hidden dir: ./.skogai/tasks
tasks: ./tasks

todo,doing,done
autocommit=true
zeropaddedids=3
defaulteditor=nvim
defaultport=3000
<!-- SECTION:DESCRIPTION:END -->

## Definition of Done
<!-- DOD:BEGIN -->
- [ ] #1 bunx tsc --noEmit passes when TypeScript touched
- [ ] #2 bun run check . passes when formatting/linting touched
- [ ] #3 bun test (or scoped test) passes
<!-- DOD:END -->
