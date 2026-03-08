# google-calendar skill (gcalcli)

User guide for the skill that manages a personal Google Calendar via `gcalcli`.

## What this skill can do

- List calendars.
- Show agenda for a date range (including TSV output for automation).
- Search events by text within a time window.
- Create events (`quick` and `add`).
- Update and delete events via `agendaupdate` (TSV).
- Preview ICS import without writing (`import --dump`).

## How to use it with Codex

- Write requests with an explicit calendar and time range.
- For changes (`create/update/delete`), include `calendar`, `time window`, and expected result.
- The skill may perform `update/delete` operations without extra confirmation prompts.

Example prompts:

- `Show 7-day agenda for calendar "Work" in TSV with id, time, title.`
- `Find events containing "sync" in "Work" over the next 30 days.`
- `Create an event tomorrow at 10:00 for 30 minutes in calendar "Work".`
- `Update the selected event title via agendaupdate and verify with search.`

## Source and Install

Install this skill in Codex:

```bash
$skill-installer install https://github.com/xmm/gcalcli-skill
```

Official project: https://github.com/insanum/gcalcli

Install with Homebrew:

```bash
brew install gcalcli
```

Install with pip (Linux):

```bash
pip install gcalcli
```

## Network and Auth

- `gcalcli` uses Google Calendar API at `https://www.googleapis.com/calendar/`.
- OAuth2 token flow uses `https://oauth2.googleapis.com/`.
- This skill is for personal calendar usage only.

## Quick requirements

- `gcalcli` must be available in `PATH`.
- `awk` must be available in `PATH` (used by TSV update pipeline examples).
- OAuth must be initialized with `gcalcli init`.

## License

This skill is distributed under the MIT license:
https://opensource.org/licenses/MIT
