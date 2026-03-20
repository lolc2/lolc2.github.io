##### Abusing Cloudflare for C2

Attackers are exploiting Cloudflare Workers, Cloudflare Tunnels, and Cloudflare Apps to establish resilient Command-and-Control (C2) infrastructures. By routing malicious traffic through Cloudflare's network, they can blend in with legitimate traffic, making detection more difficult.

#### How It Works

Cloudflare provides three primary surfaces for C2 abuse:

**Cloudflare Tunnels (`cloudflared`):** The attacker installs `cloudflared` on the compromised host, creating a tunnel that routes traffic through Cloudflare's infrastructure to the attacker's C2 server. The tunnel endpoint appears as `*.cfargotunnel.com`, making all C2 traffic look like legitimate Cloudflare usage. No inbound firewall rules are needed - the tunnel is outbound-only.

**Cloudflare Workers (`*.workers.dev`):** Serverless functions deployed on Cloudflare's edge network act as HTTP redirectors. The agent communicates with `https://<name>.workers.dev`, and the Worker forwards traffic to the actual C2 server. This is similar to Azure Functions abuse but uses Cloudflare's global CDN.

**Cloudflare Pages (`*.pages.dev`):** Static sites on Cloudflare Pages can serve as payload staging or redirect endpoints for C2 traffic.

**APT41** has been observed using Cloudflare Workers to deploy serverless code accessible via the Cloudflare CDN, effectively proxying C2 traffic.

#### Why It's Hard to Detect

- Cloudflare's domains (`cloudflare.com`, `cfargotunnel.com`, `workers.dev`, `pages.dev`) are trusted and widely used
- `cloudflared` tunnels use outbound-only HTTPS connections, bypassing inbound firewall rules
- Workers and Pages are used by millions of legitimate applications, making domain-based blocking impractical
- The attacker's actual C2 server IP is never exposed to the agent or network defenders

#### Key Detection Opportunities

1. **Unauthorized cloudflared:** The `cloudflared` binary running on hosts where Cloudflare Tunnel was not sanctioned by IT
2. **DNS monitoring:** Unusual DNS queries for `*.cfargotunnel.com` or `*.workers.dev` subdomains that don't align with business operations
3. **New endpoints:** Connections to previously unseen `*.workers.dev` or `*.pages.dev` URLs from non-browser processes
4. **User-Agent strings:** CloudflarePagesRedirector project uses default implant user-agents (`ligoloua`, `sliverua`) that serve as identifiers
