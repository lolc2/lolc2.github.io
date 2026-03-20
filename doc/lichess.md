##### Abusing Lichess for C2

Attackers can exploit Lichess's open-source chess platform API to embed commands in game moves, challenges, and streaming events, hiding C2 traffic within chess game communications.

#### How It Works

Lichess-based C2 uses the **Lichess Bot API** (`lichess.org/api/`). The attacker registers as a bot account and communicates through game moves and event streams.

**C2 Flow:** The operator challenges the agent's bot account (`POST /api/challenge/*`). The agent streams events via `GET /api/stream/event` and game data via `GET /api/bot/game/stream/`. Commands are encoded in game moves (`POST /api/bot/game/*/move/*`), and results are communicated through the game's move sequence or chat.

#### Key Detection Opportunities

1. **Process-to-domain mismatch:** Non-browser processes making API calls to `lichess.org/api/`
2. **Game move anomalies:** Bot game moves containing encoded command data or unusual patterns
3. **API token artifacts:** Lichess API tokens stored in scripts or binary strings on disk
