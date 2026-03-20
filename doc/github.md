##### Abusing GitHub for C2

Attackers can use commits, pull requests, issues, comments, gists, discussions, releases, and GitHub Actions to pass commands and exfiltrate data, hiding within the large volume of legitimate GitHub activity.

![githubc2](/doc/githubC2.png)

#### How It Works

GitHub provides multiple surfaces for C2 communication, all accessible via the GitHub REST API at `api.github.com`.

**Issue/Comment-Based C2:** The operator creates a private repository and uses issue comments as the command channel. Commands are posted as comments (optionally base64-encoded); the agent polls the issue's comment thread, retrieves new commands, executes them, and posts results as replies. The Mythic C2 framework has a dedicated GitHub profile using this pattern.

**Gist-Based C2:** GitHub Gists provide a lightweight file-sharing mechanism. The operator writes commands to a secret gist; the agent polls the gist for updates, executes commands, and writes results back by updating the gist content.

**GitHub Actions C2:** Self-hosted GitHub Actions runners can be abused as C2 execution environments. An attacker with access to a repository can trigger workflows that execute arbitrary commands on the runner machine, with results captured in workflow logs.

**Release/Raw Content:** Payloads can be hosted as GitHub releases or raw file content (`raw.githubusercontent.com`), with agents downloading staged payloads from these trusted URLs.

![githubaction](/doc/github_actions.png)

![github_comment](/doc/github_comment.png)

#### Why It's Hard to Detect

- GitHub is essential developer infrastructure - blocking `api.github.com` or `github.com` is impractical in most organizations
- The API traffic pattern (GET/POST to issues, comments, gists) is identical to legitimate developer workflows
- Private repositories provide encrypted, access-controlled communication invisible to network-level inspection
- GitHub's massive traffic volume makes anomaly detection challenging

#### Key Detection Opportunities

1. **Process-to-domain mismatch:** Non-Git/non-browser processes making authenticated API calls to `api.github.com` from non-developer workstations
2. **Token artifacts:** GitHub Personal Access Tokens (`ghp_*`) stored in scripts, binaries, or environment variables outside of standard developer tooling
3. **Polling cadence:** Regular periodic API calls to specific issue comment threads or gist endpoints at fixed intervals
4. **Encoded content:** Base64 or otherwise encoded content in issue comments, gist updates, or commit messages that doesn't match normal developer activity
5. **Actions abuse:** Unexpected workflow runs on self-hosted runners, especially those executing shell commands or downloading external payloads
