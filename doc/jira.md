##### Abusing Jira for C2

Attackers can exploit Atlassian Jira's REST API to embed commands in issue fields, comments, or attachments, disguising C2 traffic as legitimate project tracking activity.

#### How It Works

Jira-based C2 uses the **Jira REST API** (`*.atlassian.net/rest/api/2/`). The operator creates or updates Jira issues with command strings in description or comment fields. The agent polls issues via JQL queries (`/rest/api/2/search?jql=project=*`), retrieves commands, executes them, and posts results as issue comments or attachments.

#### Key Detection Opportunities

1. **Process-to-domain mismatch:** Non-browser processes making API calls to `*.atlassian.net/rest/api/2/`
2. **Polling cadence:** Periodic polling of Jira issue comments at regular intervals
3. **Token artifacts:** Jira API tokens stored in scripts or configuration files on disk
