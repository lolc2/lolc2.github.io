##### Abusing Mastodon for C2

Attackers can exploit Mastodon's decentralized social network API to post commands and retrieve results through status updates and replies, leveraging the federated nature of the platform.

#### How It Works

The [REC2](https://github.com/g0h4n/REC2) project demonstrates C2 over the **Mastodon API** (`/api/v1/statuses`). The operator posts commands as status updates on a Mastodon instance; the agent polls the status context (replies) and posts results back.

**C2 Flow:** Commands are posted via `POST /api/v1/statuses`. The agent polls `GET /api/v1/statuses/*/context` to retrieve command threads, executes instructions, and replies with output.

**Decentralized advantage:** Mastodon is federated - the C2 can use any Mastodon instance, and the operator can switch instances if one is blocked.

#### Key Detection Opportunities

1. **Process-to-domain mismatch:** Non-browser processes making API calls to any Mastodon instance `/api/v1/statuses` endpoints
2. **Polling cadence:** Regular periodic GET requests to specific status context threads
3. **Mastodon access tokens** stored in scripts or binary strings on disk
