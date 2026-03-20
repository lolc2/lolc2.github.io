##### Abusing Zulip for C2

Attackers can exploit Zulip's open-source team chat API to exchange commands through message streams, blending C2 traffic with legitimate team communication.

#### How It Works

Zulip-based C2 uses the **Zulip REST API** (`*.zulipchat.com/api/v1/`). The operator sends commands as stream messages. The agent polls for new messages, executes commands, and replies with results. File uploads via `/v1/user_uploads` enable payload delivery and data exfiltration.

**Zulip is self-hostable,** meaning attackers can run their own instance, making the C2 traffic go to a "legitimate" Zulip server.

#### Key Detection Opportunities

1. **Process-to-domain mismatch:** Non-browser processes making API calls to `*.zulipchat.com/api/v1/`
2. **Polling cadence:** Periodic message stream polling at regular intervals from automated processes
3. **API key artifacts:** Zulip API keys stored in scripts or configuration files on disk
