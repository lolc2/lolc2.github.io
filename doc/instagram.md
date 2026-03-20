##### Abusing Instagram for C2

Attackers can exploit Instagram's OAuth and Graph API to embed commands in post captions, comments, or direct messages, disguising C2 traffic as social media activity.

#### How It Works

Instagram-based C2 uses the **Instagram Graph API** (`graph.instagram.com`) and OAuth endpoints (`api.instagram.com/oauth/`). Commands can be embedded in post captions, comments, or retrieved through media metadata.

**C2 Flow:** The operator posts content or comments containing encoded commands. The agent authenticates via OAuth, polls for new content via the Graph API, retrieves commands, and posts results as comments or new content.

#### Key Detection Opportunities

1. **Process-to-domain mismatch:** Non-browser/non-Instagram processes making OAuth or Graph API calls
2. **Post content:** Captions or comments containing encoded command/response data
3. **OAuth artifacts:** Instagram API tokens stored in scripts or configuration files on disk
