##### Abusing Discord for C2

Adversaries can set up private channels or direct messages, sending commands and exfiltrating data using bot accounts or webhook integrations.

![discord](doc/discord.png)

#### How It Works

Discord-based C2 operates through two primary mechanisms: **Bot API** and **Webhooks**.

**Bot-Based C2:** The attacker creates a Discord bot via the Developer Portal, obtaining a bot token. The malware authenticates to the Discord API using this token and creates a dedicated channel (often named after the victim's hostname). Commands are sent by the operator as chat messages; the agent polls for new messages, executes them, and posts output back to the channel. Some sophisticated variants like ChaosBot (Rust-based) support commands like `shell`, `download`, and `screenshot`, posting results as file attachments.

**Webhook-Based C2:** Webhooks are write-only HTTPS endpoints that require only a URL to post data. The attacker creates a webhook in their Discord server, embeds the URL in the malware, and uses it to exfiltrate data. This is simpler than bot-based C2 but is one-directional - it's primarily used for data exfiltration and infection notifications rather than interactive command execution.

**CDN Abuse:** Discord's CDN (`cdn.discordapp.com`) is also abused to host malicious payloads, which agents download during the infection chain.

#### Why It's Hard to Detect

- Discord traffic to `discord.com` over HTTPS is common in many environments, especially those with younger or tech-oriented workforces
- Webhook URLs are write-only - defenders cannot read channel history from the URL, making retrospective investigation harder
- The webhook model requires zero infrastructure cost and zero authentication workflow beyond URL possession
- Bot traffic uses the same API endpoints as thousands of legitimate Discord bots and integrations
- Discord's TLS-protected traffic blends with legitimate usage, bypassing most perimeter filtering

#### In the Wild

Discord is one of the most heavily abused platforms for C2 across the threat landscape:

- **ChaosBot:** A Rust-based RAT that validates bot tokens via Discord API, creates per-victim channels, and provides interactive shell, download, and screenshot capabilities. Uses DLL side-loading via `identity_helper.exe`.
- **Skuld:** An open-source Go-based infostealer that exfiltrates browser credentials, cookies, and Discord tokens via webhooks.
- **LEAKGAP/Pay2Decrypt:** Ransomware variant using Discord webhooks for bot registration and post-infection C2 status updates.
- **Supply chain attacks:** Socket researchers documented malicious npm, PyPI, and RubyGems packages using hard-coded Discord webhook URLs to exfiltrate secrets from developer machines during package installation.

**Source:** [Cisco Talos - Sowing Discord](https://blog.talosintelligence.com/collab-app-abuse/), [CYFIRMA - Malicious Use of Discord](https://www.cyfirma.com/outofband/cyber-research-on-the-malicious-use-of-discord/)

#### Key Detection Opportunities

1. **Process-to-domain mismatch:** Non-Discord processes (`python.exe`, `powershell.exe`, `node.exe`, compiled binaries) making API calls to `discord.com/api/` is the strongest signal
2. **Webhook exfiltration:** POST requests to `discord.com/api/webhooks/` from any process other than a legitimate integration should be investigated
3. **IP lookup + Discord combo:** Many Discord RATs perform an external IP lookup (ipify.org) before first C2 contact - this sequence is a strong behavioral indicator
4. **Forensic artifacts:** Discord's Chromium-based cache preserves webhook URLs, attachment data, and API calls that can be recovered for forensic analysis even after content is deleted
5. **Bot token scanning:** Bot tokens in Authorization headers or embedded in binaries/scripts can be detected via pattern matching
