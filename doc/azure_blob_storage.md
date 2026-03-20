##### Abusing Azure Blob Storage for C2

Attackers can exploit Azure Blob Storage to proxy traffic into internal networks by tunneling data through blob containers. This technique turns Azure's storage endpoints into a covert channel, enabling reverse SOCKS5-like communication that can bypass egress controls.

![proxyblob socks](/doc/proxyblob_socks.png)

#### How It Works

The **proxyblob** project demonstrates how Azure Blob Storage (`*.blob.core.windows.net`) can be turned into a network tunnel.

**SOCKS Proxy Pattern:** The agent writes outbound traffic to blob objects; a relay component reads from the same blobs and forwards traffic to the destination. The reverse path works the same way, creating a bidirectional tunnel through Azure's storage infrastructure.

**Standard REST Operations:** All communication uses standard blob REST API operations (GET, PUT, HEAD, DELETE, POST) to `*.blob.core.windows.net`, making the traffic appear as normal Azure storage usage.

#### Why It's Hard to Detect

- Azure Blob Storage traffic to `*.blob.core.windows.net` is expected in environments using Azure
- The blob operations (read, write, delete) are identical to legitimate storage access
- The SOCKS proxy pattern can tunnel any protocol through the blob storage layer, including C2 traffic from other frameworks

#### Key Detection Opportunities

1. **High-frequency blob operations:** Frequent small PUT/GET requests to the same blob container from non-Azure-SDK processes
2. **SOCKS relay pattern:** Bidirectional small read/write operations at high frequency indicating tunneled traffic
3. **Encoded content:** Binary or encoded content in blob objects accessed from internal hosts that doesn't match expected storage usage
