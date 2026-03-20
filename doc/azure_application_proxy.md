##### Abusing Azure Application Proxy for C2

Attackers can leverage Azure Application Proxy (AAP) to establish a covert C2 channel by tunneling traffic through Microsoft's cloud infrastructure. AAP provides remote access to internal applications via `*.msappproxy.net`, which adversaries can exploit to proxy C2 traffic, evade network-based detections, and bypass geo-blocking or IP filtering.

![approxy](doc/appproxyc2.png)

#### How It Works

The attacker registers an Azure Application Proxy connector and configures it to relay traffic to an internal C2 endpoint. Agent traffic to `*.msappproxy.net` is forwarded through Microsoft's infrastructure to the attacker's backend.

**Advantages:** Avoids direct connections to attacker C2 servers, bypasses firewalls that whitelist Microsoft traffic, provides resilient infrastructure using Microsoft's cloud, and enables stealthy exfiltration via AAP tunneling.

**For more details:** https://www.trustedsec.com/blog/azure-application-proxy-c2

#### Key Detection Opportunities

1. **msappproxy.net traffic:** Connections to `*.msappproxy.net` from endpoints that don't normally use Azure Application Proxy
2. **Known user-agents:** AppProxyC2 uses `ApplicationProxyConnector/1.5.1975.0` and a specific Firefox 81 user-agent
3. **OAuth token monitoring:** Authentication requests to `login.microsoftonline.com` for Application Proxy resources from unexpected sources
