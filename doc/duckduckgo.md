##### Abusing DuckDuckGo for C2

Adversaries can exploit DuckDuckGo's image proxy to establish covert command-and-control (C2) channels. By embedding commands within image requests, they can disguise malicious traffic as legitimate search queries.

![duckduckgo](/doc/duckduckc2.png)

#### How It Works

DuckDuckGo's image proxy (`proxy.duckduckgo.com/iu/?u=<url>`) fetches images from arbitrary URLs on behalf of the user. The [DuckDuckC2](https://github.com/arnydo/DuckDuckC2) project exploits this by using the proxy to relay commands to an attacker-controlled server.

**C2 Flow:** The agent makes requests to DuckDuckGo's image proxy with the actual C2 server URL embedded in the `u=` parameter. DuckDuckGo fetches the URL and returns the content - which contains commands rather than images. The proxy's lack of domain allow-listing and URI sanitization enables arbitrary data relay.

#### Key Detection Opportunities

1. **Proxy abuse pattern:** Requests to `proxy.duckduckgo.com/iu/` with command parameters or attacker-controlled destination URLs
2. **Non-browser requests:** Non-browser processes making requests through the DuckDuckGo image proxy
