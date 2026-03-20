##### Abusing Slack for C2

Actors can create private channels or use Slack's file-sharing and messaging APIs to exchange commands and payloads, hidden among legitimate workspace traffic.

![slack](doc/slack.png)

#### How It Works

Slack-based C2 leverages the Slack API (Bot tokens or User tokens) to create channels, post messages, upload files, and poll for updates - all through standard Slack API endpoints at `slack.com/api/`.

**Channel-Based C2:** The operator creates a private Slack channel (or uses an existing one) as the C2 channel. Commands are posted as messages via `chat.postMessage`. The agent polls `conversations.history` at regular intervals to retrieve new commands. Execution results are posted back as replies (`conversations.replies`) or new messages. Messages can be edited (`chat.update`) or deleted (`chat.delete`) for OPSEC.

**File-Based C2:** Agents can upload and download files via `files.upload`, enabling binary payload delivery and data exfiltration through Slack's file-sharing infrastructure.

**Notable Projects:** Slackor (Coalfire) provides a full-featured C2 framework with encryption, and is one of 10+ open-source Slack C2 implementations. Mythic also has a Slack C2 profile, and F-Secure's C3 framework includes a Slack channel interface.

#### Why It's Hard to Detect

- Slack is a core business communication tool - its API traffic is expected and typically whitelisted
- Bot and User OAuth tokens (`xoxb-*`, `xoxp-*`) look identical to legitimate Slack integrations
- Private channels provide encrypted, access-controlled communication that isn't visible to workspace admins without specific audit configurations
- The polling pattern (checking for new messages) mirrors legitimate Slack bot behavior

#### Key Detection Opportunities

1. **Process-to-domain mismatch:** Non-Slack/non-browser processes making API calls to `slack.com/api/` from servers where Slack client is not installed
2. **Token artifacts:** Slack Bot tokens (`xoxb-*`) or User tokens (`xoxp-*`) stored in scripts, configuration files, or binary strings on disk
3. **Polling cadence:** Regular periodic calls to `conversations.history` or `conversations.replies` at fixed intervals from automated processes
4. **Unusual file uploads:** `files.upload` API calls from non-Slack processes containing encoded, archived, or binary payloads
