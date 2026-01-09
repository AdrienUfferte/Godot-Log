# Godot Logger (log.gd)

A lightweight logging system for Godot (GDScript) providing:
- Multiple log levels (TRACE, DEBUG, INFO, WARNING, ERROR, ALERT, PANIC)
- Automatic log file creation in `user://Logs/`
- Unified console output in Godot's Output panel
- Automatic cleanup of old log files
- Automatic log rotation when a file exceeds 5 MB
- In-memory retention of recent log entries for quick access

---

## Installation

1. Copy `log.gd` into your project (for example: `res://scripts/log.gd`).
2. Add `log.gd` as an **Autoload (Singleton)** for optimal usage:

**Godot menu**
- `Project > Project Settings > Autoload`
- Path: `res://scripts/log.gd`
- Name: `Log` (recommended)
- Click **Add**

### Why Autoload?

- Allows calling `Log.info()`, `Log.warn()`, etc. from anywhere.
- Ensures a single logger instance.
- Guarantees early initialization and consistent file handling.

---

## Log Levels and Environment Behavior

The logger automatically adapts its minimum log level depending on the runtime environment:

- **Editor**: `TRACE` (full verbosity)
- **Exported Debug build**: `DEBUG`
- **Exported Release build**: `INFO`

Logs below the current level are filtered out.

Each log line follows this format:

```
HH:MM:SS [LEVEL] message
```

Log files are created in:

```
user://Logs/
```

With the following naming scheme:

```
Kalulu_Log_YYYY-MM-DD-HH-MM-SS.txt
```

When a log file exceeds 5 MB, a new file is created using the same base name and a
rotation suffix. Examples:

```
Kalulu_Log_2024-01-31-18-42-10.txt
Kalulu_Log_2024-01-31-18-42-10_002.txt
Kalulu_Log_2024-01-31-18-42-10_003.txt
...etc...
```

Rotation starts at `_002` and continues upward. If more than 999 rotations occur,
the suffix continues as `_1000`, `_1001`, etc.

Each log file includes a note about where the next file starts and when it continues
from a previous file.

---

## API Usage

Example calls (assuming the Autoload is named `Log`):

```gdscript
Log.trace("Entering movement state")
Log.debug("Player speed = %d" % speed)
Log.info("Save loaded successfully")
Log.warn("Optional texture missing, using fallback")
Log.error("Failed to load configuration file")
Log.alert("Critical configuration missing")
Log.panic("Crash test")
```

---

## Log Output Examples

```text
12:03:11 [TRACE] Entering movement state
12:03:12 [DEBUG] Player speed = 420
12:03:13 [INFO] Save loaded successfully
12:03:14 [WARNING] Optional texture missing, using fallback
12:03:15 [ERROR] Failed to load configuration file
12:03:16 [ALERT] Critical configuration missing
12:03:17 [PANIC] Crash test
```

---

## Important Rule: DEBUG Logs Are Session-Only

`DEBUG` logs are **meant to be temporary and session-specific**.

They should:
- Never be versioned
- Never be considered long-term logs
- Only exist to track a specific issue during development

Intended workflow:
1. Add `Log.debug()` calls while investigating a bug.
2. Filter the output to show only `DEBUG` logs.
3. Focus exclusively on the logs you just added.
4. Remove them once the issue is resolved.

Permanent or meaningful information should use:
- `INFO`
- `WARNING`
- `ERROR`

---

## Filtering Logs in Godot Output

### Method A: Output Search Bar

In the **Output** panel, use the search field to filter messages.

Because logs are prefixed, filtering is straightforward:

- Show only warnings: `[WARNING]`
- Show only errors: `[ERROR]`
- Show only traces: `[TRACE]`
- Show only debug logs: `[DEBUG]`

This is especially effective when working with temporary debug logs.

### Method B: Functional Tags

You can also include functional tags in your messages:

```gdscript
Log.debug("inventory: item added %s" % item_id)
Log.error("save: failed to write slot %d" % slot)
```

Then filter using:
- `inventory:`
- `save:`

This allows precise isolation of specific systems or features.

---

## Special Levels Notes

- `ALERT`: Displays a modal dialog and blocks execution until closed.
- `PANIC`: Immediately crashes the application. Intended **only** for testing crash handlers.

---

## File Output, Rotation, and Retention Details

Each log line is written to the log file immediately and flushed for safety.

The logger keeps a rolling in-memory buffer of recent entries (about 14k lines).
When the buffer grows past the limit, the oldest entries are discarded to keep
memory usage stable.

If the log directory does not exist, it is created automatically on startup.

---

## Automatic Log Cleanup

Old log files are automatically removed at startup.

Default behavior:
- Deletes logs older than **10 days**

You can adjust this manually:

```gdscript
Log.delete_old_logs(30) # keep logs for 30 days
```
