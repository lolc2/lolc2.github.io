##### Abusing Splunk for C2

Attackers can exploit Splunk Universal Forwarder's management API and deployment server architecture to achieve remote code execution and establish a C2 channel on endpoints where Splunk agents are installed.

#### How It Works

The [SplunkWhisperer2](https://github.com/cnotin/SplunkWhisperer2) project demonstrates how Splunk Universal Forwarder (UF) misconfigurations can be exploited for local privilege escalation or remote code execution.

**The Core Vulnerability:** Splunk Universal Forwarder includes a management service listening on TCP port **8089** by default. Critically, the UF does not enforce authentication of the Deployment Server - it will accept app deployments from any server that presents itself as the DS. The default credentials are `admin/changeme`.

**Attack Flow (Local):** The attacker connects to the local Splunk UF management API on port 8089, authenticates with default or known credentials, and deploys a malicious Splunk app containing a scripted input (e.g., a `.bat` or `.ps1` file). Splunk executes this script with the privileges of the Splunk service account (often SYSTEM).

**Attack Flow (Remote via ARP Spoofing):** The attacker performs ARP spoofing to impersonate the legitimate Deployment Server, then serves a malicious app bundle to the forwarder. The UF fetches and installs the app, executing the embedded payload. After the attack, the UF automatically reconnects to the real DS and re-downloads its legitimate apps.

**Why This Is a C2 Channel:** The attacker can repeatedly deploy new scripted inputs to the forwarder to execute arbitrary commands, effectively turning the Splunk deployment infrastructure into a C2 channel. Commands are delivered as Splunk app deployments and executed by the Splunk service process.

#### Why It's Hard to Detect

- Splunk is a trusted security tool - its management traffic on port 8089 is expected and often whitelisted
- The attack uses legitimate Splunk deployment mechanisms (app installation via REST API), not exploits
- Commands execute as child processes of `splunkd`, which is a legitimate process expected to spawn scripts
- Splunk UF runs with elevated privileges (often SYSTEM) in many environments, providing immediate high-privilege access
- The malicious app is automatically cleaned up when the forwarder reconnects to the real Deployment Server

#### Key Detection Opportunities

1. **Unauthorized DS connections:** Monitor for Splunk UF deploy-poll configuration changes pointing to unknown IP addresses
2. **New app deployments:** Alert on new apps appearing in `$SPLUNK_HOME/etc/apps/` from unrecognized sources, especially those containing scripted inputs (`.bat`, `.ps1`, `.sh` in `bin/` directories)
3. **Suspicious child processes:** Monitor `splunkd.exe` / `splunkd` for spawning unexpected child processes like `cmd.exe`, `powershell.exe`, or `bash` with unusual arguments
4. **Default credentials:** Authentication to port 8089 with `admin/changeme` - should be blocked and alerted on
5. **ARP anomalies:** ARP spoofing targeting the Deployment Server IP is a precursor to remote exploitation
