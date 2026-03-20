##### Abusing SoundCloud for C2

Attackers can exploit SoundCloud's API to embed commands in track descriptions, comments, or metadata, blending C2 traffic with legitimate music streaming activity.

#### How It Works

SoundCloud-based C2 uses the **SoundCloud API** (`api-v2.soundcloud.com` / `api.soundcloud.com`). Commands can be hidden in track descriptions, comments, or custom metadata. The agent polls for updates to specific tracks and retrieves commands. Media files on `cf-media.sndcdn.com` can potentially be used for payload delivery.

#### Key Detection Opportunities

1. **Process-to-domain mismatch:** Non-browser/non-SoundCloud processes making API calls to `api-v2.soundcloud.com`
2. **Track metadata:** Track descriptions or comments containing encoded command/response data
3. **Polling cadence:** Periodic polling of specific SoundCloud resources at regular intervals
