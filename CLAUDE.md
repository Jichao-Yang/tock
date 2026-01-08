# Tock - CLI Time Tracking Tool

## ⚠️ Important: Documentation Workflow

**READ THIS FIRST:** This project uses persistent documentation to preserve knowledge across sessions. Your context is temporary—these docs are not.

### Required Documents (in `docs/`)

| Document | Purpose | When to Update |
|----------|---------|----------------|
| `docs/task_plan.md` | Track current session goals, break work into atomic tasks, record progress | **Start of session:** Define goal. **During work:** Update task status. **End of session:** Archive completed work. |
| `docs/notes.md` | Capture research findings, technical discoveries, lessons learned | **Continuously:** Whenever you learn something new about the codebase, find a bug, discover a pattern, or make a mistake worth remembering. |

### Workflow for Every Session

1. **START:** Read `docs/task_plan.md` and `docs/notes.md` to restore context
2. **PLAN:** Update `task_plan.md` with the current session's goal and task breakdown
3. **WORK:** As you implement, update task statuses in `task_plan.md`
4. **LEARN:** When you discover something useful, add it to `notes.md` immediately
5. **END:** Archive completed session in `task_plan.md`, commit docs with your changes

**Why this matters:** Information in your context window will be forgotten. These documents persist. Write things down as you go, not at the end.

---

## Overview

Tock is a command-line time tracking tool written in Go, inspired by [Bartib](https://github.com/nikolassv/bartib). It features an interactive TUI for viewing and analyzing tracked time, with a plaintext file format for storage.

## Tech Stack

- **Language:** Go 1.24
- **CLI Framework:** Cobra
- **TUI Framework:** Bubble Tea (charmbracelet)
- **Styling:** Lipgloss
- **Config:** Viper (YAML + env vars)
- **Testing:** Testify + Mockery

## Project Structure

```
cmd/tock/              # Application entry point
docs/                  # Development documentation (task_plan.md, notes.md)
internal/
├── config/            # Configuration management (Viper)
├── core/              # Domain layer
│   ├── models/        # Domain entities (Activity)
│   ├── ports/         # Interfaces (ActivityRepository, ActivityResolver)
│   ├── dto/           # Data Transfer Objects
│   └── errors/        # Domain-specific errors
├── services/          # Application layer
│   ├── activity/      # Activity operations (Start, Stop, Add, List, Report)
│   └── ics/           # iCal generation
└── adapters/          # Infrastructure layer
    ├── cli/           # CLI commands & TUI views
    ├── file/          # Flat file storage backend
    └── timewarrior/   # TimeWarrior storage backend
```

## Architecture

The project follows **Clean Architecture** (Ports & Adapters):

- **Core:** Domain models and interfaces (no external dependencies)
- **Services:** Business logic implementation
- **Adapters:** External integrations (CLI, file storage, TimeWarrior)

Dependency injection is set up in `internal/adapters/cli/root.go`.

## Key Commands

| Command | Description |
|---------|-------------|
| `start` | Begin tracking a new activity |
| `stop` | End current tracking session |
| `add` | Manually add a completed activity |
| `continue` | Resume a previous activity |
| `current` | Display currently running activity |
| `last` | List recent unique activities |
| `list` | Interactive TUI table view |
| `calendar` | Interactive calendar view (flagship) |
| `report` | Text-based reporting |
| `analyze` | Productivity analysis |
| `ical` | Export to iCal format |

## TUI Components

Located in `internal/adapters/cli/`:

- **calendar.go** - Interactive month calendar with day selection
- **calendar_sidebar.go** - Right sidebar showing selected day details
- **list_gui.go** - Table-based activity list with sorting
- **theme.go** - Color themes and styling system

## Data Format

Plaintext format (Bartib-compatible):
```
2025-12-10 09:00 - 2025-12-10 11:30 | Project Name | Task description
2025-12-10 13:00 | Another Project | Ongoing task (no end time)
```

## Configuration

- **Config file:** `~/.config/tock/tock.yaml`
- **Environment variables:** `TOCK_*` prefixed
- **Priority:** CLI flags > env vars > config file > defaults

## Development Commands

```bash
# Run tests (via Docker)
make test

# Run linter (via Docker)
make lint

# Build
go build -o tock ./cmd/tock

# Run directly
go run ./cmd/tock <command>
```

## Testing

Tests use Testify for assertions and Mockery for generating mocks. Test files:
- `internal/adapters/file/parser_test.go`
- `internal/adapters/file/repository_test.go`
- `internal/adapters/timewarrior/repository_test.go`
- `internal/services/activity/service_test.go`
- `internal/config/config_test.go`

## Code Style

- Use error wrapping with `fmt.Errorf("context: %w", err)`
- Interfaces defined in `internal/core/ports/`
- Keep adapters isolated from each other
- Follow existing patterns for new commands/views
