##### Abusing Google Drive for C2

Adversaries can hide or update payloads and fetch commands in shared or hidden folders, mixing malicious traffic with normal Google Drive operations.

![google drive](/doc/google_drive.png)

#### How It Works

Google Drive-based C2 uses the **Google Drive REST API** (`googleapis.com/drive/`) to create, read, update, and delete files that serve as the C2 message bus.

**File-Based C2:** The operator creates task files in a designated Drive folder. The agent authenticates via OAuth2 (`oauth2.googleapis.com/token`), lists folder contents via `drive/v3/files?q=...`, downloads new task files, executes commands, and uploads result files via multipart upload (`upload/drive/v3/files?uploadType=multipart`). Processed files are deleted to minimize traces.

**MIME Type Filtering:** Agents use Drive API query filters to find specific file types - for example, querying for `application/octet-stream` files in a specific folder to distinguish command files from legitimate documents.

**SOCKS Proxy (google_socks):** One implementation turns Google Drive into a SOCKS proxy, tunneling arbitrary network traffic through file read/write operations on Drive. This enables full network pivoting through Google's infrastructure.

**Notable Projects:** Rust-DriveC2 (Rust-based), GC2-sheet (combined Google Drive + Sheets), google_socks (SOCKS proxy over Drive), F-Secure C3 (GoogleDrive channel).

#### Why It's Hard to Detect

- Google Drive API traffic to `googleapis.com` is ubiquitous in Google Workspace environments
- File operations (list, upload, download, delete) mirror normal Drive sync and collaboration
- OAuth2 authentication is standard for hundreds of legitimate Google integrations
- Drive's file versioning and sharing features provide additional channels for data exchange

#### Key Detection Opportunities

1. **Process-to-domain mismatch:** Non-browser/non-Drive-client processes making API calls to `googleapis.com/drive/`
2. **Polling cadence:** Regular file listing requests to specific Drive folders at fixed intervals
3. **OAuth artifacts:** Google OAuth refresh tokens with Drive scope stored in scripts or configuration files
4. **Generic file uploads:** Files with generic or structured names uploaded via multipart API from non-browser processes
5. **SOCKS tunneling:** Unusually high volume of file read/write operations to Drive from a single process indicating traffic tunneling
