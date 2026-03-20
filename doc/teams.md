##### Abusing Microsoft Teams for C2

Attackers can exploit Microsoft Teams' chat and channel messaging to transmit commands, disguised as normal collaboration within corporate environments.

![teams](doc/teams.png)

#### How It Works

Teams-based C2 uses the **Teams Chat Service API** (`teams.microsoft.com/api/chatsvc/`). The operator sends commands as chat messages. The agent polls conversation threads for new messages, executes commands, and posts results back.

**Integration with Graph API:** Many Teams C2 implementations use the Microsoft Graph API (`graph.microsoft.com`) for broader access to Teams messages, channels, and files - see the Microsoft Graph entry for those endpoints.

#### Key Detection Opportunities

1. **Process-to-domain mismatch:** Non-Teams processes making API calls to `teams.microsoft.com/api/chatsvc/`
2. **Chat content:** Teams messages containing encoded or structured command strings from automated processes
3. **Polling cadence:** Periodic polling of Teams conversation threads at regular intervals from non-Teams applications
