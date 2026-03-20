##### Abusing Trello for C2

Attackers can exploit Trello's board, list, and card APIs to exchange commands and results through project management cards, disguising C2 traffic as legitimate productivity tool usage.

#### How It Works

Trello-based C2 uses the **Trello REST API** (`api.trello.com/1/`). The operator and agent communicate through cards on a shared Trello board.

**Setup:** The attacker creates a Trello account, obtains an API key and token, and sets up a board with designated lists (e.g., "Things To Do" for commands). The board ID and list ID are embedded in the agent.

**Command Delivery:** The operator creates cards on the board containing commands. The agent polls the board's card list via `GET /1/lists/*/cards` or `GET /1/boards/*/cards`, retrieves new cards, parses the command content, and executes it locally.

**Result Exfiltration:** The agent posts results back by creating new cards (`POST /1/cards`) or updating existing ones (`PUT /1/cards/*`). Processed cards can be deleted for OPSEC. The operator script "wipes the slate" by clearing command output after retrieval.

**Current Limitation:** The existing PoC transmits commands in plaintext over TLS. While encrypted in transit, commands and output are stored on Trello's servers unencrypted. The author notes that AES encryption for card content is planned but not yet implemented.

#### In the Wild: APT29

**APT29 (Cozy Bear / NOBELIUM)** used Trello as a C2 channel in campaigns prior to their switch to Notion. The group's **BEATDROP** malware communicated via the Trello API, creating a victim ID used to store and retrieve payloads from Trello cards. Once the victim ID was created, BEATDROP sent an initial request to Trello to check if the system was already compromised, then retrieved targeted shellcode payloads from cards. After retrieval, the payload card was deleted from Trello. APT29 later migrated this technique to Notion (GraphicalNeutrino malware), suggesting they rotate through productivity SaaS platforms for C2 to stay ahead of detection rules.

#### Why It's Hard to Detect

- Trello traffic to `api.trello.com` and `trello.com` over HTTPS is common in enterprise environments that use Trello for project management
- Card creation, reading, and deletion are standard Trello API operations identical to legitimate usage
- The API key/token authentication model is simple and looks like any other Trello integration
- APT29's documented use shows this is not just a theoretical risk - state-level actors have operationalized it

#### Key Detection Opportunities

1. **Process-to-domain mismatch:** Non-browser processes making API calls to `api.trello.com/1/` from endpoints where Trello is not a sanctioned tool
2. **Polling cadence:** Regular periodic GET requests to board/card endpoints at fixed intervals from automated processes
3. **Card lifecycle:** Rapid create-read-delete patterns on cards within a single board from non-browser processes indicate C2 command/response cycles
4. **API credential artifacts:** Trello API key and token pairs stored in scripts, configuration files, or binary strings on disk
5. **Board JSON access:** Requests to `trello.com/b/*/*.json` to retrieve full board data including list IDs - used during agent setup
