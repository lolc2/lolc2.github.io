##### Abusing Asana for C2

Attackers can exploit Asana's project management API to embed commands in tasks and retrieve results through task comments or attachments, disguising C2 traffic as legitimate project management activity.

#### How It Works

Asana-based C2 uses the **Asana API** (`app.asana.com/api/1.0/`). The operator creates tasks containing commands in a shared Asana project. The agent polls for new tasks, executes commands, and updates task status or attachments with results.

**API Endpoints Used:** Projects (`/api/1.0/projects/*`), Tasks (`/api/1.0/tasks/*`), Attachments (`/api/1.0/attachments/*`), and Sections (`/api/1.0/sections/`).

#### Key Detection Opportunities

1. **Process-to-domain mismatch:** Non-browser processes making API calls to `app.asana.com/api/`
2. **Polling cadence:** Regular polling of Asana task lists at fixed intervals from automated processes
3. **Token artifacts:** Asana Personal Access Tokens stored in scripts or configuration files on disk
