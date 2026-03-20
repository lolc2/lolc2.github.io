##### Abusing YouTube for C2

Attackers can exploit YouTube's API to embed commands in video comments, descriptions, or metadata, blending C2 traffic with the massive volume of legitimate YouTube activity.

![youtube](doc/youtube.png)

#### How It Works

YouTube-based C2 uses the **YouTube Data API** (`googleapis.com/youtube/v3/`). Commands can be hidden in video comments or descriptions. The agent polls comment threads for specific videos, retrieves encoded commands, and posts results as new comments.

**API Endpoint:** The `commentThreads` endpoint (`/youtube/v3/commentThreads?key=*`) allows reading and posting comments, providing a bidirectional communication channel.

#### Key Detection Opportunities

1. **Process-to-domain mismatch:** Non-browser processes making API calls to `googleapis.com/youtube/v3/`
2. **Comment content:** Video comments containing encoded or structured command/response data
3. **Polling cadence:** Periodic polling of specific YouTube comment threads at regular intervals
