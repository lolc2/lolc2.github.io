##### Abusing Blockchain Smart Contracts for C2

Attackers can store C2 commands, payloads, and configuration data inside public blockchain smart contracts, or use blockchain transaction data to resolve C2 infrastructure, creating channels that are permanent, decentralized, and immune to traditional takedown methods.

#### How It Works

There are three distinct patterns for blockchain-based C2, each with different network signatures:

**Pattern 1 - Storage Read (eth_call):** The attacker deploys a smart contract containing shellcode, C2 URLs, or commands in storage variables. The agent retrieves them via the `eth_call` JSON-RPC method - a read-only operation that costs nothing and creates no on-chain transaction. EtherHiding (SAAITAAMAA) demonstrates this: a Solidity contract stores raw shellcode bytes in a `bytes public shellcode` variable, and a Rust client calls `getShellcode()` via `eth_call`, then writes the returned bytes into RWX memory (VirtualAlloc with PAGE_EXECUTE_READWRITE) and executes them as a function pointer. The CLEARFAKE/EtherHiding campaigns use the same `eth_call` pattern but in browser JavaScript via the ethers.js library. This is the most common variant in the wild.

**Pattern 2 - Event-Based Bidirectional C2 (eth_sendTransaction + eth_subscribe):** The smart contract defines events that serve as C2 message channels. Both server and agent send transactions that emit events; both subscribe to the other's events via `eth_subscribe("logs")` or polling via `eth_newFilter`/`eth_getFilterChanges`. BlockchainC2 (xpn) uses this pattern with Go and the `go-ethereum` library over WebSocket (`wss://`) - the contract emits `_ServerData` and `_ClientData` events, and the agent watches for events matching its AgentID. C2-Blockchain implements the same concept in Python using `web3.py` - the server calls `tovictim(command)` to emit a `forvictim` event, and the generated virus calls `toattacker(response)` to send command output back. Both use RSA/AES or no encryption. This pattern requires gas for each message and leaves a permanent on-chain trace of all C2 traffic.

**Pattern 3 - Dead Drop Resolver via Transaction Values:** Instead of smart contracts, the attacker encodes the real C2 server address in blockchain transaction amounts. Unbreakable-Botnet-C2 (gnxbr) queries the BlockCypher REST API (`api.blockcypher.com/v1/ltc/main/addrs/`) for a Litecoin address, scans `txrefs` for a sentinel transaction value of 31337 satoshis, then concatenates the next two transaction values to construct an IP address. The bot then connects to this IP via IRC (port 6667). Glupteba pioneered this technique using Bitcoin transactions.

#### Why It's Hard to Detect

- **No server to seize:** Smart contract data is replicated across thousands of blockchain nodes worldwide
- **Immutable storage:** Once deployed, contract data cannot be deleted - the C2 infrastructure is permanent
- **Legitimate traffic blend:** `eth_call` requests to public RPC nodes look identical to normal Web3/DeFi traffic; WebSocket connections to Infura blend with dApp usage; BlockCypher API calls blend with crypto analytics
- **Zero-cost reads:** Pattern 1 (eth_call) is free, creates no transaction, and leaves no on-chain footprint
- **Multiple fallback nodes:** Dozens of public RPC endpoints exist; if one is blocked, the agent switches to another
- **Protocol diversity:** The three patterns use different protocols (HTTP POST, WebSocket, REST GET), different JSON-RPC methods, and different blockchains, making a single detection rule insufficient

#### In the Wild

Blockchain C2 has gone from theoretical PoC to active nation-state and cybercrime adoption:

- **Aeternum (2025):** Commercial botnet service using Polygon smart contracts for C2 via eth_call. Commands stored in contract annotations, operated via web panel.
- **Smargaft (2024):** Gafgyt-derived botnet using BNB Smart Chain contracts to store C2 IPs. First documented EtherHiding in the botnet field. Selects from 14 hardcoded RPC nodes.
- **CLEARFAKE/UNC5142:** 3-tier contract architecture on BSC (anchor->router->payload). AES-encrypted JS payloads via ethers.js injection in compromised WordPress sites.
- **UNC5342/DPRK (2025):** North Korean actors adopted EtherHiding for JADESNOW/INVISIBLEFERRET delivery, targeting Web3 developers.
- **SharkStealer:** Golang infostealer retrieving AES-encrypted C2 URLs from BSC Testnet contracts with hardcoded decryption key.
- **OCRFix (2026):** ClickFix botnet using BSC Testnet contracts. C2 URL rotated 5 times in 10 days. Russian-language panel.
- **Glupteba:** Pioneer - used Bitcoin transactions as backup Dead Drop Resolver for C2 addresses.

**Sources:** [Qrator Labs - Exploring Aeternum C2](https://qrator.net/blog/details/Exploring-Aeternum-C2/), [XLab - Smargaft EtherHiding](https://blog.xlab.qianxin.com/smargaft_abusing_binance-smart-contracts_en/), [Google - DPRK Adopts EtherHiding](https://cloud.google.com/blog/topics/threat-intelligence/dprk-adopts-etherhiding), [xpn - Ethereum C2](https://blog.xpnsec.com/ethereum-c2/)

#### Key Detection Opportunities

1. **No-Web3 environment rule:** In enterprises with no legitimate Web3 usage, ANY outbound connection (HTTP, WebSocket, or REST) to known blockchain RPC domains or explorer APIs is a high-confidence alert
2. **JSON-RPC method inspection:** Monitor for `eth_call`, `eth_sendTransaction`, `eth_sendRawTransaction`, `eth_subscribe`, `eth_newFilter`, and `eth_getFilterChanges` from non-browser/non-wallet processes. Pattern 1 uses `eth_call`; Pattern 2 uses `eth_sendTransaction` + `eth_subscribe` or filter polling
3. **WebSocket to RPC endpoints:** Both event-based C2 frameworks (BlockchainC2, C2-Blockchain) use `wss://` connections to Infura/public RPC nodes - monitor for WebSocket upgrades to blockchain endpoints
4. **Dead Drop chain:** HTTP GET to blockchain explorer APIs (api.blockcypher.com) followed by outbound IRC or TCP connection to a previously unseen IP is the Unbreakable-Botnet signature
5. **Memory execution after RPC:** VirtualAlloc with PAGE_EXECUTE_READWRITE followed by shellcode copy and execution after receiving data from an RPC endpoint - the EtherHiding shellcode execution pattern
6. **Library fingerprinting:** Rust `web3` crate + `winapi` memoryapi (EtherHiding); Go `go-ethereum` ethclient/bind packages (BlockchainC2); Python `web3` + `subprocess` (C2-Blockchain); C# `WebClient` + BlockCypher API + IRC socket (Unbreakable-Botnet)
7. **Testnet traffic:** Connections to Sepolia, Ropsten, Goerli, or BSC Testnet endpoints from production systems
8. **Known contract addresses and wallet addresses:** Listed in detection indicators for exact-match blocking

**Critical note:** Unlike every other LOLC2 technique, you cannot request the platform to take down the C2 data. Smart contract storage is immutable and replicated across thousands of nodes. There is no abuse report to file, no content to delete. Detection, endpoint blocking, and RPC domain filtering are the only viable defenses.
