# Task Plan

This document tracks the current implementation goals, breaks down work into atomic tasks, and records progress throughout development sessions.

---

## Current Session Goal

_Define the high-level objective for this development session._

**Session:** Interactive Calendar Planning
**Date:** 2026-01-08
**Goal:** Design and plan the "Command Mode" feature for tock calendar

---

## Feature Spec: Command Mode for Calendar

### Overview

Add vim-style `:` command mode to the existing calendar view. Press `:`, type a command, hit enter.

### Commands (MVP)

| Command | Action |
|---------|--------|
| `:start <project> <description>` | Start tracking a new activity |
| `:stop` | Stop current activity |
| `:continue` | Resume last activity (same project + description) |
| `:q` | Quit calendar |

### Behavior

- Press `:` → enter command mode, cursor appears at bottom
- Press `Enter` → execute command, return to normal mode, refresh data
- Press `Esc` → cancel command, return to normal mode
- Silent execution (no feedback messages, view updates on refresh)
- Use Bubble Tea's `textinput` component as-is

### Out of Scope (Future)

- Live auto-refresh / timer updates
- `:edit`, `:delete` commands
- Status messages / feedback

---

## Technical Implementation Plan

### Key Discovery

The `ActivityResolver` interface already exposes `Start()`, `Stop()`, and `GetRecent()` methods. The calendar's `reportModel` already holds a reference to the service. We just need to:

1. Add command mode state
2. Add text input component
3. Parse commands
4. Call existing service methods
5. Refresh data after execution

### Files to Modify

| File | Changes |
|------|---------|
| `internal/adapters/cli/calendar.go` | Add command mode state, text input, key handling, command execution |
| `go.mod` | Likely already has `bubbles/textinput` (verify) |

### Implementation Tasks

| # | Task | Status | Notes |
|---|------|--------|-------|
| 1 | Add `commandMode` state and `textinput.Model` to `reportModel` | ⏳ Pending | |
| 2 | Initialize text input in `initialReportModel()` | ⏳ Pending | |
| 3 | Handle `:` key to enter command mode | ⏳ Pending | In `handleKeyMsg()` |
| 4 | Delegate to text input in command mode | ⏳ Pending | In `Update()` |
| 5 | Handle `Enter` to execute and `Esc` to cancel | ⏳ Pending | |
| 6 | Add command parser function | ⏳ Pending | Parse `:start`, `:stop`, `:continue`, `:q` |
| 7 | Wire up `:start` to `service.Start()` | ⏳ Pending | Need `dto.StartActivityRequest` |
| 8 | Wire up `:stop` to `service.Stop()` | ⏳ Pending | |
| 9 | Wire up `:continue` using `service.GetRecent()` + `service.Start()` | ⏳ Pending | |
| 10 | Wire up `:q` to `tea.Quit` | ⏳ Pending | |
| 11 | Refresh calendar data after command execution | ⏳ Pending | Call `fetchMonthData` |
| 12 | Update `View()` to render text input at bottom when in command mode | ⏳ Pending | |
| 13 | Test manually | ⏳ Pending | |

**Status Legend:** ⏳ Pending | 🔄 In Progress | ✅ Done | ❌ Blocked | 🚫 Cancelled

---

## Technical Details

### State Changes to `reportModel`

```go
type reportModel struct {
    // ... existing fields ...

    commandMode bool              // true when in ":" command mode
    textInput   textinput.Model   // Bubble Tea text input component
}
```

### Command Parsing Logic

```go
func parseCommand(input string) (cmd string, args []string) {
    // input: "start ProjectName This is the description"
    // returns: cmd="start", args=["ProjectName", "This is the description"]

    parts := strings.SplitN(strings.TrimSpace(input), " ", 3)
    // parts[0] = command
    // parts[1] = project (for :start)
    // parts[2] = description (for :start)
}
```

### Key Flow

```
Normal Mode                    Command Mode
     │                              │
     │  press ":"                   │
     ├─────────────────────────────►│
     │                              │
     │                         type command
     │                              │
     │  press "Esc"                 │
     │◄─────────────────────────────┤
     │                              │
     │  press "Enter"               │
     │◄──────── execute ────────────┤
     │     + refresh data           │
```

---

## Backlog

_Future tasks and ideas to consider for upcoming sessions._

- [ ] Live auto-refresh while tracking (timer ticks)
- [ ] `:edit` command to modify existing activities
- [ ] `:delete` command to remove activities
- [ ] Status bar showing feedback messages
- [ ] Command history (up/down arrow)
- [ ] Tab completion for project names

---

## Completed Sessions

_Archive of past session goals for reference._

### Session: Initial Setup (2026-01-08)
- **Goal:** Set up documentation workflow
- **Outcome:** Created task_plan.md and notes.md for persistent knowledge tracking. Updated CLAUDE.md with workflow instructions for future sessions.

---

## Notes

- Keep tasks atomic and specific
- Update this document frequently during implementation
- Move completed sessions to the archive section
- Use the backlog for parking lot items
