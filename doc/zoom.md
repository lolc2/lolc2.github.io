##### Abusing Zoom for C2

Attackers can exploit Zoom's Chat API to establish a covert command-and-control channel, disguising malicious communications as normal Zoom chat activity within corporate environments.

#### How It Works

The [ShadowForgeC2](https://github.com/0xEr3bus/ShadowForgeC2) project demonstrates how Zoom's chat API can be repurposed as a C2 communication layer.

**Setup:** The attacker registers a Zoom OAuth app and obtains API credentials. The agent is configured with these credentials and targets the Zoom Chat API at `api.zoom.us/v2/chat/users/me/`.

**Command Delivery:** The operator sends commands as Zoom chat messages through the API. The agent polls the chat endpoint (`GET /v2/chat/users/me/messages`) at regular intervals to check for new instructions.

**Execution & Response:** When a new command is found, the agent executes it locally and sends the output back as a reply message via `POST /v2/chat/users/me/messages`. Messages can be edited (`PUT`) or deleted (`DELETE`) after processing for OPSEC.

**Authentication:** The agent uses OAuth tokens obtained from `zoom.us/oauth/token`, which are periodically refreshed to maintain persistent access.

#### Why It's Hard to Detect

- All traffic goes to `api.zoom.us` over HTTPS - Zoom is a sanctioned collaboration tool in most corporate environments
- The Chat API traffic patterns are indistinguishable from legitimate Zoom integrations and bots
- OAuth token-based authentication looks identical to authorized third-party Zoom apps
- Chat messages are a natural Zoom activity, providing perfect cover for encoded commands

#### Key Detection Opportunities

1. **Process-to-domain mismatch:** Non-Zoom processes making API calls to `api.zoom.us` from servers or workstations where the Zoom client is not the originating process
2. **Polling cadence:** Regular periodic GET requests to `/v2/chat/users/me/messages` at fixed intervals from automated processes
3. **OAuth token artifacts:** Zoom OAuth credentials stored in scripts or configuration files on disk
4. **Chat message content:** If decryption/inspection is available, chat messages containing structured command strings, base64 content, or unusual delimiters
