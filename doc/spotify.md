##### Abusing Spotify for C2

Attackers can exploit Spotify's Web API to hide commands within playlist descriptions, track metadata, or podcast episode notes, blending C2 traffic with legitimate music streaming.

#### How It Works

Spotify-based C2 uses the **Spotify Web API** (`api.spotify.com/v1/`). Commands can be embedded in playlist descriptions, user-created content, or episode metadata. The agent polls for updates and retrieves commands. Preview URLs (`p.scdn.co/mp3-preview/`) can potentially serve as payload delivery channels.

#### Key Detection Opportunities

1. **Process-to-domain mismatch:** Non-browser/non-Spotify processes making API calls to `api.spotify.com`
2. **Playlist anomalies:** Playlist descriptions containing encoded or structured command strings
3. **OAuth artifacts:** Spotify OAuth tokens stored in scripts or binary strings on disk
