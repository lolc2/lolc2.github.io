##### Abusing Telegram for C2

Attackers can leverage Telegram bots or private channels to send commands and exfiltrate data, blending in with normal Telegram traffic.

![telegram](doc/telegram.jpg)

#### How It Works

Telegram-based C2 operates through the **Telegram Bot API** (`api.telegram.org`). The attacker creates a bot via Telegram's @BotFather, which provides a unique API token. This token is embedded in the malware and used for all communications.

**Command Polling:** The agent calls `GET /bot<token>/getUpdates` to poll for new messages from the operator's chat. When the operator sends a command through the Telegram bot chat, the agent retrieves it, parses the instruction, and executes it locally (typically via `cmd.exe /c` or `powershell.exe`).

**Result Exfiltration:** Command output is sent back via `POST /bot<token>/sendMessage` with the chat ID and output text. For file exfiltration, the agent uses `POST /bot<token>/sendDocument` to upload stolen files (credentials, archives, screenshots) directly to the Telegram chat.

**Operator Interaction:** The operator simply uses the Telegram mobile or desktop app to issue commands and receive results, requiring zero custom infrastructure.

#### Why It's Hard to Detect

- All traffic goes to `api.telegram.org` over HTTPS (port 443), a domain trusted by virtually all corporate proxies
- Telegram is a legitimate app used by millions - blanket blocking may not be practical in every environment
- The Bot API uses standard HTTPS REST calls identical to legitimate bot integrations
- Telegram's end-to-end encryption and infrastructure obscure the true C2 server location
- Bot tokens are simple strings that can be easily obfuscated or base64-encoded in the malware binary

#### In the Wild

Telegram abuse for C2 is widespread across multiple threat actors and malware families:

- **Lazarus Group (NineRAT):** The North Korean APT deployed NineRAT, a RAT using Telegram Bot API as its C2 channel, during campaigns exploiting Log4Shell (Operation Blacksmith).
- **MuddyWater/UNC3313 (GRAMDOOR):** Iranian APT used a Python-based backdoor communicating entirely via Telegram's `sendMessage` API, with custom encoding for commands.
- **Lumma Stealer (2025):** Retrieves its actual C2 server addresses from Telegram channel names, decrypting them with ROT13/ROT15 ciphers for operational resilience.
- **DeerStealer:** Sends operator notifications via `curl.exe` POST to `/sendMessage`, alerting when a victim first executes the payload.
- **Raven Stealer:** Exfiltrates archived credential files and browser data via the `/sendDocument` endpoint.
- **Golang Backdoor (Netskope, 2025):** Copies itself to `C:\Windows\Temp\svchost.exe`, creates a bot instance via `NewBotAPIWithClient`, and supports `/cmd`, `/persist`, `/screenshot`, and `/selfdestruct` commands through Telegram chat.
- **PoshGram-C2:** A PowerShell-based Telegram C2 for Windows that uses Telegram chat as an interactive command shell, requiring no compiled binary on the target.

**Sources:** [NVISO - Telegram Abuse Detection](https://blog.nviso.eu/2025/12/16/the-detection-response-chronicles-exploring-telegram-abuse/), [Netskope - Golang Backdoor](https://www.netskope.com/blog/telegram-abused-as-c2-channel-for-new-golang-backdoor), [Mandiant - GRAMDOOR](https://cloud.google.com/blog/topics/threat-intelligence/telegram-malware-iranian-espionage)

#### Key Detection Opportunities

1. **Process-to-domain mismatch:** Any non-Telegram process (`python.exe`, `powershell.exe`, `curl.exe`, custom binaries) making requests to `api.telegram.org` is highly suspicious
2. **IP lookup + Telegram combo:** Most Telegram C2 agents perform an external IP lookup (ipify, ip-api.com) immediately before first contact with the Telegram API - this sequence is a strong behavioral indicator
3. **Bot token artifacts:** Telegram bot tokens follow the pattern `[0-9]+:[A-Za-z0-9_-]{35}` - scanning binary strings, scripts, and command lines for this pattern can reveal embedded tokens
4. **Persistent connections:** Long-duration HTTPS sessions to Telegram IP ranges (149.154.160.0/20, 91.108.4.0/22) from non-Telegram processes indicate ongoing C2 communication
5. **Where Telegram is not sanctioned:** If Telegram has no legitimate business use in your environment, any traffic to `api.telegram.org` warrants immediate investigation
