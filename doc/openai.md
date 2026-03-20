##### Abusing OpenAI for C2

Attackers can exploit OpenAI's Files API to stage commands and exfiltrate data through file upload/download operations, or use the chat completions API as a proxy for remote execution.

![openai](doc/openai1.png)

#### How It Works

OpenAI-based C2 uses the **OpenAI API** (`api.openai.com`). The Files API provides persistent storage that can serve as a C2 dead-drop.

**Files-Based C2:** The operator uploads command files via `POST /v1/files`. The agent polls `GET /v1/files/*/content` to download instructions. Results are uploaded as new files. Files on `files.oaiusercontent.com` serve as the storage layer.

**Chat Completions Proxy:** The chat API can be abused to relay instructions through AI-generated responses, effectively using OpenAI as a C2 intermediary.

#### Key Detection Opportunities

1. **Process-to-domain mismatch:** Non-browser processes making API calls to `api.openai.com` from processes other than legitimate AI tooling
2. **Files API abuse:** Upload/download activity on `files.oaiusercontent.com` from non-browser processes
3. **API key artifacts:** OpenAI API keys (`sk-*`) stored in unexpected locations outside approved AI tooling
