##### Abusing Microsoft Outlook for C2

Attackers can exploit Outlook's email drafts, messages, and home page features to establish covert C2 channels through Microsoft's enterprise email infrastructure.

![outlook](doc/azureoutlook.png)

#### How It Works

Outlook-based C2 uses two approaches:

**Draft Dead-Drop (azureOutlookC2):** Commands are placed in Outlook draft emails via the Graph API (`/me/MailFolders/drafts/messages`). The agent polls drafts, reads commands, executes them, and writes results as new drafts or messages. Since drafts are never sent, no email delivery logs are generated.

**Homepage Hijacking (Specula):** The Specula tool modifies the Outlook folder home page via registry keys, loading a malicious WebView URL whenever the user opens a specific folder. This turns Outlook itself into a persistent backdoor, with C2 commands delivered through the custom homepage.

#### Key Detection Opportunities

1. **Draft polling:** Non-Outlook processes making Graph API calls to `/me/MailFolders/drafts/messages`
2. **Homepage hijacking:** HKCU registry modifications setting custom WebView URLs for Outlook folder home pages
3. **Known indicators:** Specula uses user-agent `Specula; Microsoft Outlook`
4. **Detection patterns:** https://github.com/mthcht/ThreatHunting-Keywords/blob/main/tools/R-T/specula.csv
