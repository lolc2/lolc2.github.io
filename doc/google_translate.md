##### Abusing Google Translate for C2

Attackers can exploit Google Translate's web translation proxy to tunnel C2 traffic through Google's infrastructure, making malicious communications appear as legitimate translation requests.

#### How It Works

Google Translate can proxy arbitrary web pages via `translate.google.com/translate?u=<url>`. Attackers exploit this by hosting C2 commands on an attacker-controlled site and accessing them through Google Translate's proxy.

**BabyShark:** This C2 framework leverages Google Translate's URL rewriting to fetch commands from attacker pages via `translate.google.com/translate?&anno=2&u=<attacker-site>`. The translated page content contains embedded commands. The agent parses the commands from the proxied page and executes them locally.

**GTRS (Google Translate Reverse Shell):** Uses Google Translate as a reverse shell proxy, routing commands and responses through `*.translate.goog` domains.

**Payload Delivery:** Base64-encoded payloads can be embedded in translated page URLs, allowing the agent to download and execute staged payloads through Google's domain.

#### Why It's Hard to Detect

- All traffic goes to `translate.google.com` or `*.translate.goog` - Google-owned domains trusted by all proxies
- Translation requests are common browser activity, especially in multilingual environments
- The actual attacker-controlled content is proxied through Google's infrastructure, hiding the true destination

#### Key Detection Opportunities

1. **Process-to-domain mismatch:** Non-browser processes making requests to `translate.google.com` or `*.translate.goog`
2. **Polling pattern:** High-frequency polling of the same translated URL at regular intervals
3. **Base64 in URLs:** Translated site URLs containing base64-encoded payloads
4. **Known user-agents:** BabyShark uses a specific Chrome 70 user-agent string that is outdated and identifiable
5. **Detection blog:** https://nasbench.medium.com/understanding-detecting-c2-frameworks-babyshark-641be4595845
