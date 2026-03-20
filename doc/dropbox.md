##### Abusing Dropbox for C2

Threat actors can create or monitor specific folders or files, pushing commands and retrieving data with Dropbox's API calls, blending into legitimate file syncing.

![dropbox](/doc/dropbox.jpg)

#### How It Works

Dropbox-based C2 uses the **Dropbox API** (`api.dropboxapi.com`) to read and write files that serve as a command-and-result exchange.

**Folder-Based C2:** The operator creates a dedicated folder structure in Dropbox (e.g., `/c2/tasks/` and `/c2/results/`). Commands are written as files in the tasks folder. The agent polls via `files/list_folder`, downloads new task files via `files/download`, executes commands, and uploads results via `files/upload`. Processed task files are deleted or moved.

**Search-Based C2:** Some implementations use `files/search_v2` to look for files matching specific naming patterns, avoiding the need to poll a fixed folder path.

**Notable Projects:** DBC2 (Arno0x, Python-based with encryption), Empire (Dropbox listener with OAuth2), F-Secure C3 (Dropbox channel), pac2 (Power Automate + Dropbox combination).

#### Why It's Hard to Detect

- Dropbox traffic to `api.dropboxapi.com` and `content.dropboxapi.com` is common in many enterprises
- File sync operations (list, upload, download) are identical to the Dropbox desktop client's normal behavior
- OAuth tokens provide persistent access without requiring repeated authentication
- The agent can use standard file naming conventions to avoid content-based detection

#### Key Detection Opportunities

1. **Process-to-domain mismatch:** Non-Dropbox processes making API calls to `api.dropboxapi.com` - especially from servers where the Dropbox client is not installed
2. **Polling cadence:** Regular `list_folder` calls at fixed intervals from automated processes
3. **Token artifacts:** Dropbox OAuth tokens or API keys in scripts, environment variables, or binary strings
4. **Encoded uploads:** File uploads to `content.dropboxapi.com/2/files/upload` containing encoded or archived payloads from non-Dropbox processes
5. **Known user-agents:** Empire's Dropbox listener uses a distinctive IE11 user-agent string
