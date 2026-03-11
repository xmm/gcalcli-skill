---
name: google-calendar
description: Manage a personal Google Calendar via gcalcli. Use when tasks involve list/search agenda, creating events, updating or deleting by TSV, and importing ICS with parse-friendly non-interactive commands.
---

# gcalcli Skill

Prerequisites: `gcalcli` and `awk` in `PATH`, wrapper scripts at `scripts/gcalcli-update` and `scripts/gcalcli-delete`, and OAuth initialized via `gcalcli init`.

User-facing setup and usage notes: see `README.md`.

## Security and Privacy (Strict)

- STRICTLY FORBIDDEN: read, print, copy, parse, or exfiltrate OAuth tokens/refresh tokens/client secrets from `gcalcli` config, cache, or auth files.
- STRICTLY FORBIDDEN: run commands that dump raw token material from local auth files.
- If authentication is broken, use `gcalcli init`; do not inspect token files.

## Core Rules

- Use `gcalcli` for read/search/create operations.
- Use stable wrappers for update/delete operations:
  - `scripts/gcalcli-update --calendar "CALENDAR_NAME" ...`
  - `scripts/gcalcli-delete --calendar "CALENDAR_NAME" ...`
- These wrappers keep a stable command prefix for Codex allow-rules.
- `scripts/gcalcli-update` and `scripts/gcalcli-delete` always add `--nocolor` internally.
- Prefer wrapper inline-flags mode for mutations (no external stdin payload).
- TSV passthrough for wrappers is opt-in via `--from-stdin`.
- If you need `agendaupdate`-specific options, pass them after `--`.
- Prefer non-interactive commands.
- Prefer simple output: use `--tsv` and global `--nocolor` where supported.
- Always pass global flags before subcommands:
  - `gcalcli --nocolor list`
  - `gcalcli --calendar "CALENDAR_NAME" agenda ...`
- For write operations, always pass global `--calendar` explicitly (for example `"CALENDAR_NAME"`).
- `--details` must be repeated (`--details id --details title`), not comma-separated.
- Available event-output fields for `--details` (canonical order):
  - `id,time,url,conference,title,location,description,calendar,email,action,length,reminders,attendees,attachments,end,all`

## Avoid By Default

- `gcalcli edit` (interactive)
- `gcalcli delete` (interactive)
- `gcalcli config edit` (opens editor)

## Parse-Friendly Read Patterns

Use one command at a time:

List calendars (use this first to get exact names for `--calendar`):

```bash
gcalcli --nocolor list
```

Agenda across all calendars:

```bash
gcalcli --nocolor agenda \
  --tsv \
  --details id \
  --details time \
  --details title \
  --details calendar \
  "today" "7 days"
```

Agenda for one calendar:

```bash
gcalcli --nocolor --calendar "CALENDAR_NAME" agenda \
  --tsv \
  --details id \
  --details time \
  --details title \
  "today" "7 days"
```

Search in one calendar:

```bash
gcalcli --nocolor --calendar "CALENDAR_NAME" search \
  --tsv \
  --details id \
  --details time \
  --details title \
  "sync" "today" "30 days"
```

## Non-Interactive Write Patterns

### Create Event (quick parser)

```bash
gcalcli --nocolor --calendar "CALENDAR_NAME" quick \
  --details id \
  --details time \
  --details title \
  "Skill test event tomorrow 10am"
```

### Create Event (explicit fields)

```bash
gcalcli --nocolor --calendar "CALENDAR_NAME" add \
  --title "Skill test event" \
  --when "tomorrow 10:00" \
  --duration 30 \
  --description "Created by gcalcli skill" \
  --noprompt \
  --details id \
  --details time \
  --details title
```

### Create Event (with reminders)

```bash
gcalcli --nocolor --calendar "CALENDAR_NAME" add \
  --title "Skill test event with reminders" \
  --when "tomorrow 10:00" \
  --duration 30 \
  --reminder "30 popup" \
  --reminder "1h email" \
  --noprompt \
  --details id \
  --details time \
  --details title
```

### Update Event Start Time (preferred, no stdin payload)

```bash
scripts/gcalcli-update --calendar "CALENDAR_NAME" \
  --event-id "EVENT_ID" \
  --start-date "2026-03-12" \
  --start-time "11:30" \
  --end-date "2026-03-12" \
  --end-time "12:00"
```

### Update Event with arbitrary fields (preferred, no stdin payload)

```bash
scripts/gcalcli-update --calendar "CALENDAR_NAME" \
  --event-id "EVENT_ID" \
  --set title="Updated title" \
  --set location="Room 301"
```

### Delete Event by ID (non-interactive, preferred)

```bash
scripts/gcalcli-delete --calendar "CALENDAR_NAME" --event-id "EVENT_ID"
```

### Delete Event via TSV (bulk optional)

```bash
printf 'action\tid\ndelete\tEVENT_ID\n' \
| scripts/gcalcli-delete --calendar "CALENDAR_NAME" --from-stdin
```

### Import Preview Without Writing

```bash
cat <<'ICS' | gcalcli --calendar "CALENDAR_NAME" import --dump
BEGIN:VCALENDAR
VERSION:2.0
BEGIN:VEVENT
UID:test-1
DTSTAMP:20260306T120000Z
DTSTART:20260307T100000Z
DTEND:20260307T103000Z
SUMMARY:Test Event
END:VEVENT
END:VCALENDAR
ICS
```

## Notes

- `agenda` shows all events in a time window.
- `search` filters events by text query (`text`) in a time window.
- `scripts/gcalcli-update` and `scripts/gcalcli-delete` wrap `gcalcli --nocolor ... agendaupdate`.
- For `scripts/gcalcli-update`/`scripts/gcalcli-delete`, pass exactly one calendar via global `--calendar`.
- Prefer inline flags mode for wrappers to avoid external stdin payloads.
- Use `--from-stdin` only when intentionally applying TSV input.
- `import` can read ICS from `stdin` (no file argument needed).
- For reliable patch/delete through TSV, include the `id` column.
- TSV piping is optional and useful mainly for bulk edits.
- Keep a temp file only when a human needs to manually inspect/edit TSV before apply.
- Always verify mutations with `gcalcli --nocolor --calendar "CALENDAR_NAME" search ... --tsv`.
