##### Abusing CounterStrike 1.6 for C2

Threat actors can use game servers, chat features, or modded RCON protocols to tunnel commands and data under the guise of normal gameplay.

![counterstrike](doc/counterstrike.jpg)

#### How It Works

The [1.6-C2](https://github.com/eversinc33/1.6-C2) project exploits the **Source RCON (Remote Console) Protocol** used by Counter-Strike 1.6 game servers. RCON allows authenticated remote administration of game servers - the attacker repurposes this mechanism as a C2 channel.

**C2 Flow:** The agent connects to an attacker-controlled CS 1.6 server on UDP port 27015 (the default game port). Commands are embedded in RCON protocol messages. The agent executes them and sends results back through the same protocol channel, disguised as normal game server communication.

#### Why It's Hard to Detect

- CS 1.6 server traffic on UDP 27015 may be dismissed as gaming activity, especially in environments where gaming traffic isn't explicitly monitored
- The RCON protocol is a legitimate administration feature, not an exploit
- UDP-based communication doesn't maintain persistent TCP connections that are easier to flag

#### Key Detection Opportunities

1. **Unusual UDP traffic:** Outbound UDP traffic on port 27015 from workstations or servers where CS 1.6 is not installed
2. **RCON protocol analysis:** Source RCON protocol commands from unexpected source IPs or to non-gaming endpoints
3. **Environmental context:** Game server traffic from corporate endpoints is itself an anomaly in most environments
