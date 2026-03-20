##### Abusing Reddit for C2

Attackers can abuse Reddit's API to establish covert C2 channels by using subreddit posts and comments as a message bus for command delivery and result exfiltration.

![reddit](doc/reddit.png)

#### How It Works

The [RedditC2](https://github.com/kleiton0x00/RedditC2) project demonstrates how Reddit's comment system becomes a bidirectional C2 channel. The operator and agent communicate entirely through comments on a specific Reddit post within an attacker-controlled subreddit.

**Setup:** The operator creates a Reddit account, registers an OAuth app (obtaining a `client_id` and `secret`), and creates a subreddit. A post within this subreddit serves as the "listener" - the shared communication channel.

**Command Delivery:** The operator posts a comment on the listener post prefixed with `in:` followed by the command to execute. The agent periodically polls the post's comment thread via the Reddit API, looking for new comments containing the `in:` prefix.

**Execution & Response:** When the agent finds a new command, it parses the comment, executes the command locally, and replies to the comment with the output prefixed with `out:`. The original command comment is then edited to `executed` to avoid re-execution.

**OPSEC:** RedditC2 supports encrypted C2 traffic and a stealth mode that deletes C2 logs after execution, leaving minimal forensic traces on the Reddit platform itself.

#### Why It's Hard to Detect

- All traffic goes to `reddit.com` / `oauth.reddit.com` over HTTPS - a domain that is extremely common in corporate environments
- Reddit API calls look identical to legitimate Reddit browsing or third-party app usage
- Comments on subreddit posts are a natural Reddit activity, making content-based detection very difficult
- The attacker can use private or low-visibility subreddits to minimize exposure
- As the author notes: "since most of the blue-team members use Reddit, it might be a great way to make the traffic look legit"

#### Key Detection Opportunities

1. **Process-to-domain mismatch:** Non-browser processes (python.exe, custom binaries) making authenticated API calls to `oauth.reddit.com` should be investigated
2. **Polling cadence:** Regular GET requests to the same Reddit comment thread at fixed intervals is inconsistent with normal browsing behavior
3. **OAuth credential artifacts:** Reddit OAuth `client_id` and `secret` stored in config files (e.g., `config.json`) on disk indicate programmatic Reddit access
4. **Comment pattern analysis:** If network inspection is available, comments following a structured format (`in:`, `out:`, `executed`) are strong indicators
