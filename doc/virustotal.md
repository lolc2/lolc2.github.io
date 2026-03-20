##### Abusing VirusTotal for C2

Attackers can abuse VirusTotal's comment system to store commands and exchange data, hiding C2 communications within a platform trusted by security teams themselves.

#### How It Works

VirusTotal-based C2 exploits the **comment functionality** on file and URL analysis entries via the VT API (`virustotal.com/api/`).

**Comment-Based C2:** The operator uploads a benign file or URL to VirusTotal, creating a persistent analysis object. Commands are posted as comments on this object via `POST /api/v3/files/<hash>/comments`. The agent polls the same object's comments via `GET /api/v3/files/<hash>/comments`, retrieves new commands, executes them, and posts results as new comments. Commands and results can be encoded (base64, hex, or custom encoding) to avoid detection by VT's content monitoring.

**Why VirusTotal Is Particularly Ironic:** Security analysts routinely browse VirusTotal, making traffic to `virustotal.com` not only whitelisted but actively expected from security-focused workstations. The platform is literally designed for security teams - using it as a C2 channel exploits the trust defenders place in their own tools.

**Notable Projects:** VirusTotalC2 (multiple implementations), REC2 (multi-platform C2 supporting VT and Mastodon), SharpHungarian.

#### Why It's Hard to Detect

- VirusTotal is a security tool - its traffic is expected and trusted, especially from SOC analyst workstations
- The API is commonly used by automated security tools (SIEM integrations, sandbox detonation, IOC enrichment)
- Comments on VT objects are a legitimate feature, and there's no content-based restriction that would flag encoded strings
- API keys are commonly distributed across security tooling, making their presence on disk unremarkable

#### Key Detection Opportunities

1. **Process-to-domain mismatch:** Non-security-tool processes making authenticated API calls to `virustotal.com/api/` should be investigated
2. **Comment polling pattern:** Regular periodic GET requests to a specific file or URL's comments endpoint at fixed intervals indicates C2 polling
3. **Comment content anomalies:** Comments containing base64, hex-encoded, or structured command strings rather than legitimate analysis notes
4. **API key location:** VT API keys (64-character hex strings) found in unexpected locations - outside of approved security tooling configurations
