##### Abusing Mattermost for C2

Attackers can exploit Mattermost's self-hosted messaging platform API to exchange commands through channels and posts, blending C2 traffic with legitimate team communication.

#### How It Works

Mattermost-based C2 uses the **Mattermost REST API** (`/api/v4/`). The operator creates channels and posts commands as messages. The agent polls channel post history, retrieves commands, executes them, and posts results as replies. File uploads and thread replies provide additional communication channels.

**API Endpoints:** The comprehensive API covers users (`/api/v4/users/*`), posts (`/api/v4/posts/*`), channels (`/api/v4/channels`), teams (`/api/v4/teams`), and file links (`/api/v4/files/*/link`). Posts can be patched (`/api/v4/posts/*/patch`) to update command status.

**Self-Hosted Advantage:** Since Mattermost can be self-hosted, the attacker can run their own instance, making the C2 server a legitimate Mattermost deployment.

#### Key Detection Opportunities

1. **Process-to-domain mismatch:** Non-browser processes making API calls to `*/api/v4/` on a Mattermost instance
2. **Polling cadence:** Regular polling of channel posts (`/api/v4/channels/*/posts`) at fixed intervals
3. **Token artifacts:** Mattermost Personal Access Tokens stored in scripts or binary strings on disk
4. **Self-hosted instances:** Outbound connections to unknown Mattermost servers not sanctioned by the organization
