##### Abusing Matrix for C2

Attackers can exploit the Matrix decentralized messaging protocol to send commands through encrypted room messages, leveraging the federated and end-to-end encrypted nature of the platform.

#### How It Works

Matrix-based C2 uses the **Matrix Client API** (`matrix.org/_matrix/client/r0/`). Commands are posted as room messages; the agent polls the room for new messages and replies with execution results.

**C2 Flow:** The operator posts commands via `POST /_matrix/client/r0/rooms/*/send/m.room.message`. The agent polls with `GET /_matrix/client/r0/rooms/*/messages`, retrieves commands, executes them, and posts results back to the room.

**E2E Encryption:** Matrix supports end-to-end encrypted rooms, making content inspection impossible at the network level. The federated nature means the C2 can use any Matrix homeserver.

#### Key Detection Opportunities

1. **Process-to-domain mismatch:** Non-Element/non-Matrix-client processes making API calls to `matrix.org/_matrix/client/`
2. **Polling cadence:** Periodic room message polling at regular intervals from automated processes
3. **Access token artifacts:** Matrix access tokens stored in scripts or binary strings on disk
4. **Self-hosted homeservers:** Connections to unknown Matrix homeservers not sanctioned by the organization
