##### Abusing Notion for C2

Attackers can exploit Notion's API to store commands and exfiltrate data through pages and blocks, making malicious traffic appear as legitimate note-taking activity.

#### How It Works

The [OffensiveNotion](https://github.com/mttaggart/OffensiveNotion) project demonstrates a full-featured C2 agent that uses Notion as its communication backend. Notion pages and their block children serve as the message bus between operator and agent.

**Setup:** The attacker creates a Notion integration via `notion.so/my-integrations`, obtaining an API token (prefixed with `secret_`). A Notion page is shared with this integration and serves as the C2 channel. The page ID and token are embedded in the agent binary.

**Command Delivery:** The operator writes commands into Notion page blocks (paragraphs, code blocks, or database entries) via the Notion API or directly through the Notion UI. The agent polls `GET /v1/blocks/<page_id>/children` at regular intervals to check for new instructions.

**Execution & Response:** The agent parses command blocks, executes them locally, and writes results back by updating blocks via `PATCH /v1/blocks/<block_id>` or creating new child blocks. The Notion page becomes a live, bidirectional C2 console.

**Advanced Features (OffensiveNotion):** The agent supports shell command execution, file download/upload, port scanning, privilege escalation, and persistence - all orchestrated through Notion page interactions. It's written in Rust and compiles to native binaries for Windows, Linux, and macOS.

**Mythic Integration (0xbbuddha/notion):** A Mythic C2 profile that uses Notion databases as the transport. Each message is a database page with payloads stored in code blocks (avoiding Notion's 2000-character property limit). Processed pages are automatically archived for OPSEC.

**Hog C2:** Disguised as "FocusForge," a legitimate-looking productivity tracker built with Tauri (Rust + WebView). The application actually provides real time-tracking features as cover, while its Rust backend communicates with Notion API for C2 - making it indistinguishable from a real productivity tool.

#### In the Wild: APT29

**APT29 (Cozy Bear)** has been observed abusing Notion API for C2 in a campaign targeting the European Commission (2022). The GraphicalNeutrino malware used Notion as its C2 channel, embedding commands in the Notion workspace accessed via the API. This followed the group's earlier use of Trello for C2 (BEATDROP malware), demonstrating a pattern of abusing productivity platforms. APT29 has historically used Twitter, GitHub, and cloud storage services for C2, and Notion represents their expansion into productivity SaaS abuse.

**Source:** [Gianluca Tiepolo - APT29 Campaign Abuses Notion API](https://mrtiepolo.medium.com/sophisticated-apt29-campaign-abuses-notion-api-to-target-the-european-commission-200188059f58)

#### Why It's Hard to Detect

- All traffic goes to `api.notion.com` over HTTPS - Notion is a widely used productivity tool trusted by corporate proxies
- The API calls (`GET /v1/pages`, `PATCH /v1/blocks`) are identical to legitimate Notion integrations and automations
- Notion is commonly used for documentation, project management, and knowledge bases - its traffic blends perfectly into enterprise environments
- The API token format (`secret_*`) is a simple string easily embedded in binaries
- The operator can interact with the C2 channel through Notion's normal web or desktop UI, leaving no custom infrastructure footprint

#### Key Detection Opportunities

1. **Process-to-domain mismatch:** Non-Notion processes (especially compiled binaries, python, or scripts) making API calls to `api.notion.com` should be investigated
2. **Polling cadence:** Regular GET requests to `/v1/blocks/*/children` or `/v1/pages/*` at fixed intervals from non-browser processes is characteristic of C2 beaconing
3. **Bidirectional block updates:** Alternating GET (read commands) → PATCH (write results) patterns on the same Notion page indicate interactive C2 communication
4. **Token artifacts:** Notion API integration tokens (`secret_*`) stored in binary strings, environment variables, or configuration files on disk
5. **Cross-platform agents:** OffensiveNotion compiles for Windows, Linux, and macOS - detection rules should cover all three platforms making requests to `api.notion.com`
