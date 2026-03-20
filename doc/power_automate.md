##### Abusing Microsoft Power Automate for C2

Attackers can exploit Power Automate's flow automation capabilities to create C2 channels that leverage Microsoft's cloud infrastructure, integrating with services like Teams, Dropbox, and SharePoint for command delivery and data exfiltration.

#### How It Works

The [pac2](https://github.com/NTT-Security-Japan/pac2) project demonstrates C2 over Power Automate. The attacker creates automation flows that act as command relays between cloud services and compromised endpoints.

**C2 Flow:** The attacker creates Power Automate flows connecting to Teams, Dropbox, or other connectors. Commands are sent via Teams messages or Dropbox files. The flow processes them and delivers instructions to the agent. Results are routed back through the same flow, leveraging Power Automate as an orchestration layer.

**Integration Points:** The pac2 framework connects through Teams (`/api/teams/messages`), Dropbox (`content.dropboxapi.com`), and Power Automate's own management API (`providers/Microsoft.PowerApps/apis/shared_flowmanagement`).

#### Why It's Hard to Detect

- Power Automate is a legitimate Microsoft enterprise tool - its traffic to `make.powerautomate.com` is expected
- The flows use authorized connectors with legitimate OAuth tokens
- Commands traverse through multiple Microsoft services, making single-point detection difficult
- The `azure-logic-apps/1.0` user-agent is a legitimate Power Automate identifier

#### Key Detection Opportunities

1. **Unauthorized flows:** New Power Automate flows created with connections to external services not sanctioned by IT
2. **Flow management API:** Non-browser processes making API calls to `make.powerautomate.com` or `flow.microsoft.com`
3. **Cross-service traffic:** Unusual combination of Teams + Dropbox API calls from the same flow or session
