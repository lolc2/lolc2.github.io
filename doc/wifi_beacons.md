##### Abusing Wi-Fi Beacons for C2

Attackers can exploit 802.11 Wi-Fi beacon frames to create a covert, air-gap-crossing C2 channel by encoding commands in SSID fields and vendor-specific Information Elements - requiring no network association or internet connectivity.

#### How It Works

The [WIFIAIR-C2-Channel](https://github.com/V-i-x-x/WIFIAIR-C2-Channel) project demonstrates a unidirectional C2 transport using Wi-Fi beacon SSID fields.

**RF-Based C2:** Unlike all other LOLC2 techniques that abuse cloud APIs, this approach operates at the physical layer - 802.11 radio frames. The operator broadcasts Wi-Fi beacon frames with commands encoded in the SSID field (up to 32 bytes per beacon). The agent, running on a system with a Wi-Fi adapter, passively scans for these beacons and extracts commands from the SSID data.

**No Network Association Required:** The agent never associates with any Wi-Fi network. It simply performs active scans using the OS's native Wi-Fi API (e.g., `WlanScan` on Windows) and reads the SSID fields from detected beacon frames. This means the C2 channel works even on air-gapped systems that have a Wi-Fi adapter but no network connectivity.

**Unidirectional Channel:** The current PoC is unidirectional - commands flow from operator to agent via beacons. The agent cannot easily send responses back through the same channel without additional hardware, though probe request frames could theoretically be used for the upstream direction.

**Data Encoding:** Commands are fragmented across multiple beacon frames if they exceed the 32-byte SSID limit. Vendor-specific Information Elements in the beacon frame provide additional capacity for encoding data. The channel is low-bandwidth (constrained by scan intervals of ~4 seconds per cycle) but sufficient for short commands.

#### Why It's Hard to Detect

- **No network traffic:** There are no TCP/IP connections, no DNS queries, no HTTPS requests - traditional network monitoring sees nothing
- **No internet required:** Works on air-gapped systems or systems with disabled network interfaces, as long as the Wi-Fi radio is physically present
- **Passive reception:** The agent only scans for Wi-Fi networks (a normal OS behavior) - it doesn't transmit data or associate with any network
- **RF-only:** Detection requires wireless IDS/IPS or RF monitoring equipment, which many organizations don't deploy
- **Plausible deniability:** Wi-Fi scanning is a normal OS behavior that occurs continuously on most systems

#### Key Detection Opportunities

1. **Unusual SSID patterns:** Wireless IDS detecting beacon frames with SSID fields containing base64, hex, or non-human-readable strings
2. **Rapid SSID changes:** Beacons from the same BSSID with rapidly changing SSIDs indicate data is being encoded in the SSID field
3. **High-frequency scanning:** Endpoint processes calling `WlanScan` or equivalent at abnormally high frequency (every few seconds) without user interaction
4. **Rogue transmitters:** RF monitoring detecting beacon frames from unauthorized transmitters in physically controlled environments (SCIFs, secure labs)
5. **Physical security:** This attack requires the operator's transmitter to be within Wi-Fi range of the target - physical access controls are the primary defense
