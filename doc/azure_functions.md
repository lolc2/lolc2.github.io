##### Abusing Microsoft Azure Functions for C2

Attackers can set up serverless Azure Functions to relay commands or stage payloads, leveraging legitimate cloud endpoints to avoid raising suspicion.

![azurefunctions](/doc/azurefunctions.png)

#### How It Works

Azure Functions-based C2 exploits serverless compute to create **HTTP redirectors** - lightweight proxies that relay C2 traffic between the agent and the actual C2 server, making the traffic appear to originate from legitimate Azure infrastructure.

**HTTP Redirector Pattern:** The attacker deploys an Azure Function app with HTTP-triggered functions. The agent communicates with `https://<app-name>.azurewebsites.net/api/<function-name>`. The function receives the agent's beacon, forwards it to the real C2 server (Cobalt Strike, Sliver, etc.), and relays the response back. To defenders, all traffic appears to go to a trusted `*.azurewebsites.net` domain.

**FunctionalC2 (Red Siege):** Provides dedicated Azure Function endpoints for C2 operations - `*/FortyNorth/PostIt` for agent check-ins, `*/FortyNorth/GetIt` for command retrieval, and `*/FortyNorth/StageIt` for payload staging. The function acts as a transparent proxy between the agent and the backend C2 server.

**AzureC2Relay (Flangvik):** A more generic Azure Function relay that can proxy traffic for any HTTP-based C2 framework, making it framework-agnostic.

**AzureFunctionRedirector:** Specifically designed to front Cobalt Strike traffic through Azure Functions, with configurable routing rules.

#### Why It's Hard to Detect

- All agent traffic goes to `*.azurewebsites.net` - a Microsoft-owned domain hosting millions of legitimate applications
- Azure Functions are widely used for legitimate serverless APIs, webhooks, and integrations
- The attacker's actual C2 server IP is never exposed to the agent or network defenders
- Azure Functions can be deployed and torn down rapidly, enabling infrastructure rotation
- The traffic pattern (HTTPS POST/GET to an API endpoint) is indistinguishable from legitimate Azure Function usage

#### Key Detection Opportunities

1. **New Azure endpoints:** Alert on outbound connections to previously unseen `*.azurewebsites.net/api/*` endpoints, especially from non-browser processes
2. **Relay behavior:** Traffic pattern analysis may reveal the Azure Function acting as a pass-through - response timing and payload characteristics may match known C2 frameworks
3. **User-Agent anomalies:** Specific user-agents like `Microsoft-CryptoAPI/6.1` in requests to Azure Function endpoints can indicate C2 agent traffic
4. **Payload inspection:** Azure Function responses containing encoded or binary payloads (shellcode, beacon configurations) inconsistent with legitimate API responses
5. **Domain age/reputation:** Newly created Azure Function apps with no prior history receiving traffic from internal hosts warrant investigation
