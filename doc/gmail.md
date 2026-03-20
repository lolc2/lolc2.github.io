##### Abusing Gmail for C2

Attackers can send commands and retrieve data via email messages or drafts, leveraging Google's APIs and IMAP/SMTP protocols for stealthy command communication.

#### How It Works

Gmail-based C2 uses two primary approaches: the **Gmail REST API** and **IMAP/SMTP protocols**.

**Draft Dead-Drop (gcat pattern):** The most common technique uses Gmail drafts as a dead-drop channel. The operator writes a command in a draft email (never sent). The agent authenticates to Gmail via OAuth or App Password, polls the drafts folder, reads the command, executes it, and writes the output as a new draft. The command draft is then deleted. Since no email is ever sent, there are no sender/receiver logs in email security gateways.

**Message-Based C2:** The agent monitors the inbox for emails matching specific criteria (subject line, label, or sender). Commands are embedded in the email body. After execution, results are sent back as a reply or new email. This approach is less stealthy than drafts since emails traverse Google's mail infrastructure.

**IMAP/SMTP Direct:** Some implementations connect directly to `imap.gmail.com:993` and `smtp.gmail.com:465`, bypassing the REST API entirely. This uses standard email protocols, making the traffic appear as a normal email client.

**Notable Projects:** gcat (Python, draft-based), gmailc2 (Python), SharpGmailC2 (.NET, inbox polling), PSGSHELL (PowerShell), GmailBackdoor.

#### Why It's Hard to Detect

- Gmail traffic to `googleapis.com` and `gmail.com` over HTTPS is ubiquitous in enterprise environments
- The draft dead-drop technique generates no sent/received email logs - traditional email security gateways see nothing
- OAuth authentication looks identical to legitimate Google Workspace integrations
- IMAP/SMTP connections to Gmail are expected from email clients

#### Key Detection Opportunities

1. **Process-to-domain mismatch:** Non-email-client processes (python.exe, custom binaries) authenticating to Gmail API or connecting to imap.gmail.com
2. **Draft polling pattern:** Repeated API calls to `/gmail/v1/users/me/drafts` at regular intervals from non-browser processes is highly suspicious
3. **OAuth credential artifacts:** Google OAuth refresh tokens or Gmail App Passwords stored in scripts or configuration files on disk
4. **DNS anomalies:** Unexpected spikes in DNS queries for `imap.gmail.com` or `smtp.gmail.com` from workstations that don't normally use IMAP
