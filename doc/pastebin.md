##### Abusing Pastebin for C2

Attackers can store commands and payloads in Pastebin pastes, using the platform as a dead-drop for C2 communication that blends with developer and IT activity.

#### How It Works

Pastebin-based C2 uses the **Pastebin API** (`pastebin.com/api/`) and raw paste access (`pastebin.com/raw/*`).

**C2 Flow:** The operator creates pastes containing commands via `POST /api/api_post.php`. The agent periodically fetches specific paste URLs via `pastebin.com/raw/*` to retrieve commands. Results can be posted back as new pastes. Pastes can be set to private or unlisted to limit visibility.

**Common Pattern:** Many malware families use Pastebin as a first-stage payload host or configuration retrieval point rather than full interactive C2, downloading encoded payloads or C2 server addresses from raw paste URLs.

#### Key Detection Opportunities

1. **Process-to-domain mismatch:** Non-browser processes fetching content from `pastebin.com/raw/` or making API calls to `pastebin.com/api/`
2. **Polling cadence:** Periodic fetching of specific Pastebin paste URLs at regular intervals
3. **API key artifacts:** Pastebin developer keys stored in scripts or binary strings on disk
