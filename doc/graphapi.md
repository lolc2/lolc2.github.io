##### Abusing Microsoft Graph API for C2

Adversaries can exploit the Microsoft Graph API **`https://graph.microsoft.com`** to establish covert command-and-control (C2) channels by leveraging legitimate Microsoft 365 services such as OneDrive, SharePoint, Outlook, Teams, and Excel Online. By embedding malicious communications within these cloud applications, attackers evade traditional security controls and persist within an enterprise environment.

#### How It Works

Microsoft Graph is the single unified REST API to all of Microsoft 365. One authenticated HTTP request can read email, list user accounts, access files, query device inventory, or pull security alert data. Three OAuth flows produce Graph tokens, each with a different trust model and blast radius:

**Client Credentials Flow** (`grant_type=client_credentials`): No user, no browser, no MFA. An application authenticates directly using a client_id and client_secret (or certificate) to obtain a token scoped to application-level permissions. This is the most dangerous flow for C2 - the token is indistinguishable from legitimate application traffic and generates only non-interactive sign-in events that most SOC teams never see.

**Authorization Code Flow**: Standard interactive user authentication producing access and refresh tokens. The refresh token is the prize - it outlives the session, survives password resets in some configurations, and can silently re-issue access tokens for up to 90 days.

**Device Code Flow** (`grant_type=urn:ietf:params:oauth:grant-type:device_code`): Designed for input-constrained devices but repurposed for phishing. The attacker generates a real device code, sends the legitimate microsoft.com/devicelogin URL to the victim, who completes real MFA on a real Microsoft page and unknowingly hands the attacker a fully authenticated token.

#### Commonly Abused Endpoints

**Reconnaissance:** `/v1.0/users` (full tenant directory dump), `/v1.0/groups` (org chart mapping), `/v1.0/servicePrincipals` (privilege mapping - identifies which apps have broad permissions and exposed secrets), `/beta/identity/conditionalAccess/policies` (maps defensive gaps before data access).

**Email Access & Impersonation:** `/v1.0/users/{id}/messages` (read any mailbox with Mail.Read application permission), `/v1.0/users/{id}/sendMail` (send email as any user - passes SPF/DKIM/DMARC from legitimate mailbox), `/v1.0/me/MailFolders/drafts/messages` (C2 via draft emails - FINALDRAFT, AzureOutlookC2).

**File Access & Exfiltration:** `/v1.0/me/drive/root/children`, `/v1.0/sites/*/drives`, `/v1.0/sites/*/lists/*` - OneDrive and SharePoint access for payload staging and data exfiltration.

**Persistence:** `/v1.0/applications/*/addPassword` (add client secret to existing app), `/v1.0/applications/*/addKey` (upload certificate), `/v1.0/oAuth2PermissionGrants` (consent grant manipulation).

**Teams C2:** `/beta/*metadata#chats`, `/beta/*metadata#teams*/channels*/messages` - commands disguised as normal chat messages.

#### Why It's Hard to Detect

- **No endpoint to compromise:** Graph-based attacks are API calls to a Microsoft-hosted service. No malware, no lateral movement, no endpoint artifact
- **Identical to legitimate traffic:** The API call structure from a C2 implant is identical to any legitimate Graph integration. The API returns 200 OK regardless of how the token was obtained
- **Non-interactive sign-in blindspot:** Client credentials token activity ONLY appears in non-interactive sign-in logs - most SOC teams monitor only interactive sign-ins
- **Token is the intrusion:** A stolen refresh token bypasses MFA entirely since authentication already completed. Infostealers (Redline, Lumma, Raccoon) harvest these at industrial scale
- **Ubiquitous domain:** `graph.microsoft.com` is on every enterprise allowlist

#### Key Detection Opportunities

1. **Non-interactive sign-in logs:** Client credentials tokens generate non-interactive sign-in events only. Monitor for service principal sign-ins outside business hours, from unexpected IPs, or with atypical API call sequences
2. **Device code flow:** Look for `deviceCode` grant type in Entra ID sign-in logs with unfamiliar application IDs - legitimate device code usage in enterprise is narrow and predictable
3. **Dangerous permission combinations:** Service principals holding Mail.Read + Mail.Send, Files.ReadWrite.All, or User.Read.All at application level warrant immediate scrutiny
4. **Graph Activity Logs:** `MicrosoftGraphActivityLogs` (must be explicitly enabled via Azure Monitor) records ALL HTTP requests to Graph - the critical data source for detecting bulk enumeration and cross-mailbox access
5. **Credential persistence operations:** POST to `/applications/*/addPassword` or `/applications/*/addKey` from unexpected identities indicates attacker adding persistence
6. **Draft email C2:** Draft message creation/modification not associated with normal user Outlook activity
7. **Cross-tenant token abuse:** Token refresh activity referencing a client_id that does not belong to your organization

**Sources:** [GraphStrike Developer Blog](https://redsiege.com/blog/2024/01/graphstrike-developer/), [Microsoft Graph API Attack Surface (Skliar, 2026)](https://infosecwriteups.com/microsoft-graph-api-attack-surface-oauth-flows-abused-endpoints-and-what-defenders-miss-9c303ea2aa02), [Detecting Threats with Graph Activity Logs (Palo Alto)](https://www.paloaltonetworks.com/blog/security-operations/detecting-threats-with-microsoft-graph-activity-logs/), [A History of GraphAPI Attacks](https://mattysplo.it/2025/02/07/graphapi.html)

![graphstrike](/doc/graphstrike.png)

#### Known Graph API C2 Malware Families

The following malware families have been observed using Microsoft Graph API for C2 in real-world campaigns, demonstrating the breadth of nation-state and cybercrime adoption:

- **FINALDRAFT (2025):** C++ backdoor targeting foreign ministries (REF7707/CL-STA-0049). Uses Outlook drafts folder via Graph API for bidirectional C2 - commands received as drafts, results written as new drafts. 37 command handlers including process injection and PowerShell execution without powershell.exe. Linux variant also exists.
- **Havoc/SharePoint C2 (2025):** Modified Havoc framework delivered via ClickFix phishing. Uses SharePoint via Graph API as FUD C2 - all stages hidden behind SharePoint, traffic indistinguishable from file collaboration.
- **SIESTAGRAPH (2023):** Predecessor to FINALDRAFT, Graph API C2 via Outlook mail service.
- **GraphStrike (2024):** Red team tool enabling Cobalt Strike Beacons to use Graph API for C2 via SharePoint.
- **BirdyClient:** OneDrive-based C2 via Graph API, involves vxdiff.dll and apoint.exe process artifacts.
- **Graphite/Graphican:** Nation-state backdoors using OneDrive/SharePoint via Graph API.
- **Bluelight:** Exploits CVE-2021-40444, uses OneDrive via Graph API for C2.

#### Detection Data Sources

Two critical log sources for Graph API threat hunting:

- **MicrosoftGraphActivityLogs** (Sentinel/Log Analytics): Records ALL HTTP requests to Graph API. Must be explicitly enabled via Azure Monitor diagnostic settings. Cost-dependent.
- **GraphApiAuditEvents** (Defender XDR Advanced Hunting): Free alternative released July 2025 in public preview. Captures Graph API requests including read operations that were invisible in standard audit logs. Recommended starting point for organizations without Sentinel budget.

Both support KQL queries for detecting AzureHound/GraphRunner enumeration patterns, bulk mail access, document exfiltration, and OAuth credential manipulation.
