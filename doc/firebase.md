##### Abusing Firebase Realtime Database for C2

Firebase Realtime Database can be abused as a cloud-hosted message bus to queue commands, exchange results, and move files via structured database entries, blending C2 traffic into a widely trusted Google cloud service. This removes the need for a dedicated C2 server and can create near real-time bidirectional tasking.

![FireC2 Schema](doc/firec2_schema.svg)

#### How It Works

The [firec2](https://github.com/s0ld13rr/firec2) project demonstrates a complete C2 architecture built entirely on Firebase Realtime Database as the communication layer. No dedicated C2 server is required - Firebase acts as a shared message queue between the operator console and the agent.

**Agent Registration:** On first run, the agent generates a random 6-character ID and registers itself by writing host metadata (OS, hostname, username, architecture, IP, PID) to a Firebase path `agents/<agent_id>/meta.json` using an HTTP PUT request.

**Task Polling:** The agent polls the Firebase path `tasks/<agent_id>.json` using GET requests at randomized intervals (10–30 seconds). When the operator issues a command via the console, it writes a task entry with an HMAC signature to this path. The agent verifies the signature using a shared secret before executing.

**Command Execution:** The agent executes shell commands via `subprocess` with `shell=True`, then writes stdout/stderr results back to `results/<agent_id>/<task_id>.json`. Large outputs are gzip-compressed and base64-encoded before upload.

**File Transfer:** File exfiltration is done by reading the target file, base64-encoding its content, computing a SHA256 hash for integrity, and writing the entire payload to `files/<agent_id>/<filepath>.json`. File downloads from operator to agent use the same pattern through the `uploads/` path.

**Heartbeat:** A PATCH request updates `agents/<agent_id>/heartbeat.json` every 60 seconds with a `last_seen` timestamp, allowing the operator to track active agents.

All communication uses standard HTTPS REST API calls to `*.firebaseio.com` with JSON payloads and a `.json` suffix on all paths. This makes the traffic appear as normal Firebase application usage, blending into legitimate Google cloud traffic.

#### Why It's Hard to Detect

- All traffic goes to `*.firebaseio.com` over HTTPS (port 443), a domain trusted by virtually all corporate proxies and firewalls
- Firebase Realtime Database is widely used by legitimate mobile and web applications, making blanket domain blocking impractical
- The REST API pattern (`GET/PUT/PATCH` to `*.firebaseio.com/*.json`) is identical to how legitimate apps interact with Firebase
- No custom domains, no custom protocols, no unusual ports - only standard HTTPS to Google infrastructure
- The agent uses a common browser User-Agent string, making traffic inspection harder
- WebSocket upgrades to Firebase domains (used by the real-time sync protocol) are also expected in legitimate usage

#### In the Wild: FireScam

The **FireScam** malware (late 2024) demonstrated real-world abuse of Firebase Realtime Database as a C2 channel. Disguised as a fake "Telegram Premium" app, it used Firebase RTDB to exfiltrate stolen data including messages, notifications, clipboard contents, and screen activity. The exfiltrated data was temporarily stored at `androidscamru-default-rtdb.firebaseio.com` before being filtered and moved to private storage. FireScam also used WebSocket connections to Firebase for real-time bidirectional C2 communication, making its traffic blend with legitimate Firebase app traffic.

**Source:** [CYFIRMA - Inside FireScam: An Information Stealer with Spyware Capabilities](https://www.cyfirma.com/research/inside-firescam-an-information-stealer-with-spyware-capabilities/)

#### Key Detection Opportunities

1. **Process-to-domain mismatch:** Non-browser processes (`python.exe`, `powershell.exe`, custom binaries) making HTTPS requests to `*.firebaseio.com` - legitimate Firebase usage is almost exclusively from mobile apps or web browsers
2. **Polling cadence:** Regular GET requests to the same `*.firebaseio.com` path at fixed or slightly randomized intervals are characteristic of C2 beaconing, unlike event-driven legitimate app usage
3. **JSON path structure:** The `.json` suffix on all REST API paths is a Firebase-specific pattern - monitoring for paths like `/tasks/`, `/results/`, `/agents/`, `/files/` can reveal C2-specific naming conventions
4. **Large PUT payloads:** Base64-encoded file contents written to Firebase database paths produce unusually large JSON payloads that differ from normal application data writes
5. **Configuration artifacts:** The agent stores its Firebase URL as a base64-encoded string in the binary/script, which can be detected via static analysis or YARA rules scanning for base64-decoded `firebaseio.com` patterns
