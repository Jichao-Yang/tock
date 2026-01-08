# Development Notes

This document captures research findings, technical discoveries, lessons learned, and other knowledge gained during development. Use this to persist information across sessions so it doesn't get lost in context.

---

## Project Understanding

### Architecture Overview (2026-01-08)

Tock follows Clean Architecture (Ports & Adapters):

```
┌─────────────────────────────────────────────────────────────┐
│                        Adapters                              │
│  ┌─────────┐  ┌──────────────┐  ┌────────────────────────┐  │
│  │   CLI   │  │     File     │  │      TimeWarrior       │  │
│  │ (Cobra) │  │  Repository  │  │      Repository        │  │
│  └────┬────┘  └──────┬───────┘  └───────────┬────────────┘  │
└───────┼──────────────┼──────────────────────┼───────────────┘
        │              │                      │
        ▼              ▼                      ▼
┌─────────────────────────────────────────────────────────────┐
│                       Services                               │
│  ┌─────────────────────────────────────────────────────┐    │
│  │              ActivityService                         │    │
│  │   (Start, Stop, Add, List, Report, Continue)        │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
        │
        ▼
┌─────────────────────────────────────────────────────────────┐
│                         Core                                 │
│  ┌──────────┐  ┌───────────┐  ┌──────────┐  ┌──────────┐   │
│  │  Models  │  │   Ports   │  │   DTOs   │  │  Errors  │   │
│  │(Activity)│  │(Interfaces)│  │          │  │          │   │
│  └──────────┘  └───────────┘  └──────────┘  └──────────┘   │
└─────────────────────────────────────────────────────────────┘
```

**Key insight:** Dependency injection happens in `internal/adapters/cli/root.go`. New features should follow this pattern.

### TUI Framework Notes

- **Bubble Tea** uses the Elm architecture: Model → Update → View
- All TUI components are in `internal/adapters/cli/`
- Calendar view is the flagship feature (~478 lines)
- Theme system supports 256-color and 16-color ANSI modes

### Data Storage

Plaintext format (Bartib-compatible):
```
2025-12-10 09:00 - 2025-12-10 11:30 | Project Name | Task description
```
- Ongoing activities have no end time
- File is human-readable and editable
- Default location: `~/.tock.txt`

---

## Research Findings

_Document findings from code exploration, external research, etc._

### Bubble Tea Patterns Used (2026-01-08)

From examining the codebase:
- `tea.Model` interface implemented by calendar, list views
- `tea.Cmd` for async operations
- `tea.KeyMsg` for keyboard handling
- `lipgloss` for all styling (colors, borders, layout)

### Command Mode Implementation Analysis (2026-01-08)

**Key discovery:** The `ActivityResolver` interface already has all the methods we need:
- `Start(ctx, dto.StartActivityRequest)` - start new activity
- `Stop(ctx, dto.StopActivityRequest)` - stop current activity
- `GetRecent(ctx, limit)` - get recent activities (for `:continue`)

The calendar's `reportModel` already holds a `service ports.ActivityResolver`, so we can directly call these methods without any DI changes.

**Calendar model structure** (`internal/adapters/cli/calendar.go`):
```go
type reportModel struct {
    service      ports.ActivityResolver  // ← Already has Start/Stop!
    currentDate  time.Time
    viewDate     time.Time
    monthReports map[int]*dto.Report
    viewport     viewport.Model
    ready        bool
    width, height int
    err          error
    styles       Styles
    theme        Theme
}
```

**Key methods to understand:**
- `handleKeyMsg()` - handles all keyboard input, add `:` case here
- `Update()` - main Bubble Tea update loop
- `View()` - renders the UI, need to add command input at bottom
- `fetchMonthData()` - returns `tea.Cmd` that fetches data, call after commands

**DTOs needed for commands:**
```go
// For :start
dto.StartActivityRequest{
    Description: "...",
    Project:     "...",
    StartTime:   time.Time{}, // empty = now
}

// For :stop
dto.StopActivityRequest{
    EndTime: time.Time{}, // empty = now
}
```

### Bubble Tea Text Input Component

Need to import: `github.com/charmbracelet/bubbles/textinput`

Basic usage:
```go
ti := textinput.New()
ti.Placeholder = "Enter command..."
ti.Focus()

// In Update():
ti, cmd = ti.Update(msg)

// In View():
ti.View()
```

---

## Lessons Learned

_Capture what worked, what didn't, and insights for future sessions._

### Session: 2026-01-08

1. **Project is mature** - Most core features already implemented, focus on enhancements
2. **Clean separation** - Adding new features means touching adapters/cli for commands, services for logic
3. **Testing setup** - Uses Docker for isolated test/lint runs via Makefile

---

## Technical Debt & Observations

_Note areas that could be improved or require attention._

- _Add observations here as they arise during development_

---

## Quick Reference

### Useful Commands
```bash
# Build and run
go build -o tock ./cmd/tock && ./tock <command>

# Run directly
go run ./cmd/tock <command>

# Test (Docker)
make test

# Lint (Docker)
make lint
```

### Key Files
| Purpose | Location |
|---------|----------|
| Entry point | `cmd/tock/main.go` |
| DI setup | `internal/adapters/cli/root.go` |
| Calendar TUI | `internal/adapters/cli/calendar.go` |
| Activity service | `internal/services/activity/service.go` |
| Domain model | `internal/core/models/activity.go` |
| File parser | `internal/adapters/file/parser.go` |
