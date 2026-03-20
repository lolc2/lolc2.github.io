##### Abusing OneDrive for C2

OneDrive can be used to upload, update, and retrieve payloads or exfiltrate data through shared documents or hidden folders, blending in with normal cloud usage.

#### How It Works

OneDrive-based C2 operates through the **Microsoft Graph API** (`graph.microsoft.com`), using OneDrive file operations as the communication layer.

**File-Based C2:** The operator creates files in a specific OneDrive folder (e.g., `tasks/` for commands, `results/` for output). The agent polls the folder via `GET /v1.0/me/drive/root/children`, downloads new task files, executes the commands, and uploads result files. Files can be created, read, updated, and deleted through standard Graph API calls.

**Authentication:** Agents authenticate using OAuth2 tokens obtained from `login.windows.net` or `login.microsoftonline.com`. Empire's OneDrive listener uses this approach with long-lived refresh tokens for persistence.

**Notable Projects:** Empire (OneDrive listener), GraphStrike (Cobalt Strike over Graph API), Excel-C2 (uses Excel files in OneDrive as command channel), F-Secure C3 (OneDrive365RestFile channel).

#### Why It's Hard to Detect

- Microsoft Graph API calls to `graph.microsoft.com` are standard enterprise traffic in Microsoft 365 environments
- OneDrive file operations are indistinguishable from legitimate document sync and collaboration
- OAuth tokens for Microsoft Graph are routinely used by hundreds of legitimate applications
- The same Graph API is used by OneDrive, SharePoint, Teams, and Outlook - C2 traffic blends with all of them

#### Key Detection Opportunities

1. **Process-to-domain mismatch:** Non-Office/non-browser processes making Graph API calls to `/me/drive/` endpoints
2. **Polling cadence:** Regular periodic file listing or download from a specific OneDrive folder at fixed intervals
3. **Unusual file operations:** Large or encoded file uploads to OneDrive from non-Office applications
4. **Token artifacts:** OAuth tokens with `Files.ReadWrite` scope stored in scripts or configuration files
5. **Known user-agents:** Empire's OneDrive listener uses the user-agent `Microsoft SkyDriveSync 17.005.0107.0008`
