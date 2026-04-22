##### Abusing chess.com for C2

Attackers can abuse **Chess.com Game Collections and the Analysis workflow** as a covert transport layer by storing tasking and output inside imported chess records instead of using the site for normal gameplay.

#### How It Works

This implementation does **not** rely on playing live games or sending normal moves to an opponent. It uses authenticated requests to the **Chess.com collection callback endpoints** to upload, list, and remove imported games tied to specific collection IDs.

The transport logic works in several stages:

1. Raw task/output bytes are first **Base64-encoded**.
2. The Base64 bytes are converted into a custom **Base5 alphabet** using only `P`, `N`, `B`, `R`, and `Q`.
3. That encoded stream is split into **8-character chunks** and packed into **FEN board rows**.
4. Up to **six FEN rows** are stored per record, with short rows padded using chess digits.
5. Each FEN is wrapped inside a **PGN** using `[SetUp "1"]` and `[FEN "..."]` headers.
6. The generated PGN batch is uploaded into a Chess.com collection through `.../actions/add-from-pgn`.
7. The peer polls the paired collection via `.../items`, reconstructs the FEN sequence, strips digit padding, decodes the custom Base5 alphabet, and recovers the original data.

The code also uses a fixed **sentinel FEN** (`7k/8/8/8/8/8/8/7K w - - 0 1`) as a marker before the actual payload sequence. After processing, collection entries are deleted through `.../actions/remove-items`.

#### Why It Matters

This kind of channel blends into traffic for a well-known consumer platform while abusing a legitimate feature: **importing and storing PGN/FEN-based analysis content in collections**. From a detection point of view, the valuable signal is not “Chess traffic” in general, but **browser-like authenticated requests to collection callback endpoints from non-browser tooling**, combined with repeated PGN/FEN import behavior.

#### Key Detection Opportunities

1. **Collection callback endpoint abuse:** direct requests to `/callback/library/collections/` from scripts, implants, or unusual processes.
2. **PGN/FEN import anomalies:** repeated uploads containing `[SetUp "1"]` and dense FEN payloads rather than normal user-generated analysis.
3. **Marker/sentinel reuse:** repeated appearance of `7k/8/8/8/8/8/8/7K w - - 0 1` around collection operations.
4. **Token misuse:** Chess.com session cookies and collection upload/clear tokens used outside an interactive browser session.
5. **Bulk collection churn:** fast cycles of upload → poll → remove against the same collection IDs.
