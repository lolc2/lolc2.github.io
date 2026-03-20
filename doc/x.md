##### Abusing X for C2

Attackers can leverage the X (formerly Twitter) API to send commands through tweets or DMs and exfiltrate data via replies, DMs, or bio updates, blending malicious traffic with legitimate social media activity.

#### How It Works

X-based C2 uses the **X API v2** (`api.x.com/2/`). The operator posts commands as tweets or sends them via DMs. The agent polls for new tweets or DM messages, executes commands, and posts results back.

**Command Channels:** Tweets (`/2/tweets`), Direct Messages (`/2/dm_conversations/with/*/messages`), user bio updates (`/2/users/me`), and Lists (`/2/lists`) can all serve as C2 communication surfaces.

**Limitation:** The free API tier restricts to GET requests for posts, limiting interactive C2 without a paid plan.

#### Why It's Hard to Detect

- X/Twitter API traffic is common from social media management tools and integrations
- DMs provide an encrypted, private channel for C2 communication
- Bio updates provide a one-to-many broadcast mechanism for commands

#### Key Detection Opportunities

1. **Process-to-domain mismatch:** Non-browser/non-Twitter processes making API calls to `api.x.com`
2. **Polling cadence:** High-frequency GETs to `/users/<id>/tweets` or `/dm_conversations` at regular intervals
3. **Structured bio updates:** PATCH requests to `/users/me` with structured or encoded bio content
4. **API credential artifacts:** X API Bearer tokens or OAuth credentials stored in scripts or binary strings
