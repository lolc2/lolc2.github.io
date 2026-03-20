##### Abusing Twitter for C2

Attackers can leverage Twitter's legacy API (v1.1) to send commands through tweets, DMs, or media uploads, blending C2 traffic with legitimate social media activity.

#### How It Works

Twitter-based C2 uses the **Twitter API v1.x and v2** to exchange commands and data. The operator posts commands as tweets or sends them via DMs. The agent polls for new content, executes commands, and replies with results. Media uploads (`upload.twitter.com`) can be used for file exfiltration.

**Note:** With the migration to the X API (v2), some legacy Twitter C2 tools may need updated endpoints. See the X entry for v2-specific techniques.

#### Key Detection Opportunities

1. **Process-to-domain mismatch:** Non-browser processes making API calls to `api.twitter.com`
2. **OAuth artifacts:** Twitter API credentials stored in scripts or binary strings on disk
3. **Polling cadence:** Periodic polling of tweet threads or DMs at regular intervals
