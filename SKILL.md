---
name: google-calendar
description: Manage a personal Google Calendar via gcalcli. Use when tasks involve list/search agenda, creating events, updating or deleting by TSV, and importing ICS with parse-friendly non-interactive commands.
---

# gcalcli Skill

Prerequisites: `gcalcli` and `awk` in `PATH`, and OAuth initialized via `gcalcli init`.

User-facing setup and usage notes: see `README.md`.

## Security and Privacy (Strict)

- STRICTLY FORBIDDEN: read, print, copy, parse, or exfiltrate OAuth tokens/refresh tokens/client secrets from `gcalcli` config, cache, or auth files.
- STRICTLY FORBIDDEN: run commands that dump raw token material from local auth files.
- If authentication is broken, use `gcalcli init`; do not inspect token files.

## Core Rules

- Use `gcalcli` command name.
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

### Update Event via TSV (`patch` default action)

```bash
gcalcli --nocolor --calendar "CALENDAR_NAME" agenda \
  --tsv \
  --details id \
  --details time \
  --details title \
  "today" "2 days" \
| awk 'BEGIN{FS=OFS="\t"} NR==1{for(i=1;i<=NF;i++) if($i=="title") t=i; print; next} {if(t) $t=$t" [UPDATED]"; print}' \
| gcalcli --calendar "CALENDAR_NAME" agendaupdate
```

### Delete Event via TSV (non-interactive)

```bash
printf 'action\tid\ndelete\tEVENT_ID\n' \
| gcalcli --calendar "CALENDAR_NAME" agendaupdate
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
- `agendaupdate` requires exactly one calendar via global `--calendar`.
- `import` can read ICS from `stdin` (no file argument needed).
- For reliable patch/delete through TSV, include the `id` column.
- Prefer pipe for generated TSV payloads (fewer temp files, simpler automation).
- Keep a temp file only when a human needs to manually inspect/edit TSV before apply.
- Always verify mutations with `gcalcli --nocolor --calendar "CALENDAR_NAME" search ... --tsv`.
