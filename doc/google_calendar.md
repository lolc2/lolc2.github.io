##### Abusing Google Calendar for C2

Operators can embed commands in event descriptions or notes, checking calendar entries via the Calendar API to retrieve instructions or exfiltrate data.

![google calendar](doc/google_calendar.png)

#### How It Works

Google Calendar-based C2 uses the **Calendar API** (`calendar.googleapis.com`). Commands are embedded in calendar event descriptions, and the agent polls for new or updated events.

**C2 Flow:** The operator creates or updates calendar events with commands in the description field via `POST/GET /calendar/v3/calendars/*/events`. The agent polls for events, parses commands from descriptions, executes them, and writes results as new events or event updates.

**APT41** has been observed using Google Calendar for C2 coordination, embedding C2 instructions in calendar event data.

![google calendar apt41 sample](doc/google_calendar_apt41_example.png)

![google calendar apt41 sample 2](doc/google_calendar_campaign.jpg)

#### Key Detection Opportunities

1. **Process-to-domain mismatch:** Non-browser processes making API calls to `calendar.googleapis.com` - especially suspicious tooling user-agents (Python, curl, Go-http-client)
2. **Calendar event content:** Events with encoded or structured command strings in description fields
3. **Process indicators:** `python.exe` making direct outbound connections to `calendar.googleapis.com`, or rare binaries contacting `*.googleapis.com` with `calendar/v3` path
