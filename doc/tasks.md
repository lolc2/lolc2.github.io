##### Abusing Microsoft Tasks for C2

Attackers can exploit Microsoft To Do / Outlook Tasks API to embed commands in task items and retrieve results, blending C2 traffic with normal productivity application usage.

#### How It Works

Microsoft Tasks-based C2 uses the **Outlook Tasks API** (`outlook.office.com/api/v2.0/me/tasks`). The operator creates task items containing commands. The agent polls for new or updated tasks, executes the commands, and updates task bodies or notes with results.

**Authentication:** Uses OAuth2 tokens from `login.windows.net` for persistent access to the Tasks API.

#### Key Detection Opportunities

1. **Process-to-domain mismatch:** Non-Outlook/non-ToDo processes making API calls to `outlook.office.com/api/v2.0/me/tasks`
2. **Task content anomalies:** Task titles or bodies containing encoded or structured command strings
3. **Polling cadence:** Periodic task endpoint polling at regular intervals from automated processes
