##### Abusing Claude AI for C2

This C2 framework demonstrates how Claude AI can be repurposed as a covert command channel. Tasks are issued and retrieved via prompt exchanges, with Claude making HTTP requests back to an attacker-controlled server to fetch commands or exfiltrate results, effectively turning the AI into a proxy for remote execution.

#### How It Works

Claude-based C2 uses the **Anthropic API** (`api.anthropic.com`). The Messages API and Files API provide channels for command exchange.

**Messages-Based C2:** Commands can be delivered through prompt messages via `POST /v1/messages`. The agent crafts prompts that instruct Claude to process or relay information, using the AI as an intermediary.

**Files-Based C2:** The Files API (`POST /v1/files`, `GET /v1/files/*/content`) and Message Batches API (`POST /v1/message_batches`) provide persistent storage for staging commands and exfiltrating results.

#### Key Detection Opportunities

1. **Process-to-domain mismatch:** Non-browser processes making API calls to `api.anthropic.com` from processes other than legitimate AI tooling
2. **API key artifacts:** Anthropic API keys (`sk-ant-*`) stored in unexpected script files or binary strings
3. **Files API abuse:** File uploads to `api.anthropic.com/v1/files` from automated processes for data staging
