##### Abusing Cisco Webex for C2

Attackers can exploit Webex's messaging and rooms API to exchange commands and payloads, hidden within legitimate enterprise collaboration traffic.

#### How It Works

Webex-based C2 uses the **Webex API** (`webexapis.com/v1/`). The operator creates a Webex room and sends commands as messages. The agent polls the room for new messages, executes commands, and posts results back. File attachments can be used for payload delivery and data exfiltration.

**API Endpoints:** Access tokens (`/v1/access_token`), messages (`/v1/messages`), rooms (`/v1/rooms`), and message retrieval with room and pagination filters.

#### Key Detection Opportunities

1. **Process-to-domain mismatch:** Non-Webex processes making API calls to `webexapis.com/v1/`
2. **Polling cadence:** Periodic polling of `/v1/messages` at regular intervals from non-browser processes
3. **Bot token artifacts:** Webex Bot access tokens stored in scripts or binary strings on disk
