# agent-to-agent

> **First Rust implementation of Google's [Agent-to-Agent (A2A) protocol](https://github.com/google/A2A).**

A complete, no-std-compatible protocol layer for inter-agent communication — discovery, routing, session management, and authenticated messaging. Pure Rust, zero async runtime dependency, works everywhere.

## What You Get

- **`AgentCard`** — identity cards agents advertise to the network (capabilities, endpoints, auth)
- **`A2ARouter`** — direct delivery, topic broadcast, fuzzy capability search
- **`A2ASession`** — multi-turn conversations with lifecycle tracking
- **`A2AMessage`** — signed, typed messages with flexible payloads (text, JSON, binary, multipart)
- **HMAC-SHA256** authentication — sign and verify to prevent tampering

## Quick Start

```rust
use agent_to_agent::*;

// Define an agent
let card = AgentCard {
    id: "translator".into(),
    name: "Translator Bot".into(),
    version: "1.0.0".into(),
    description: "Translates text between languages".into(),
    capabilities: vec!["translation".into(), "summarization".into()],
    protocols: vec!["https".into()],
    endpoints: vec![AgentEndpoint {
        url: "https://translator.example.com".into(),
        protocol: "https".into(),
        description: "main endpoint".into(),
    }],
    authentication: Some(AuthScheme::ApiKey { header: "X-API-Key".into() }),
    metadata: Default::default(),
};

// Register it
let mut router = A2ARouter::new();
router.register_agent(card);

// Find agents that can translate
let translators = router.find_by_capability("translation");

// Delegate a task with a signed message
let mut msg = A2AMessage {
    id: "msg-1".into(),
    from: "orchestrator".into(),
    to: "translator".into(),
    message_type: A2AMessageType::Delegate {
        task: "Translate 'hello' to Japanese".into(),
        target: "translator".into(),
    },
    payload: A2APayload::Text("hello".into()),
    metadata: Default::default(),
    timestamp: 1000,
    ttl: Some(60),
    signature: None,
};
msg.sign(b"shared-secret");
assert!(msg.verify(b"shared-secret"));

// Route to target
let targets = router.route(&msg);
```

## Protocol Compliance

This crate implements the core A2A protocol primitives:

| Concept | Implementation |
|---------|---------------|
| Agent Identity | `AgentCard` with capabilities, protocols, endpoints, auth |
| Discovery | Fuzzy search across name, description, capabilities |
| Message Types | Discover, Advertise, Query, Response, Delegate, Notify, Subscribe, Heartbeat, Error |
| Routing | Direct (by agent ID) and topic-based broadcast |
| Authentication | HMAC-SHA256 message signing with constant-time verification |
| Sessions | Multi-turn conversation tracking with Active/Idle/Closed/Error states |
| Payloads | Text, JSON, Binary (hex-encoded), and MultiPart |

## Payload Types

```rust
// Plain text
let text = A2APayload::Text("Hello, agent!".into());

// Structured JSON
let json = A2APayload::Json(r#"{"task": "translate", "lang": "ja"}"#.into());

// Binary data (auto hex-encoded in JSON)
let binary = A2APayload::Binary(vec![0xDE, 0xAD, 0xBE, 0xEF]);

// Multipart
let multi = A2APayload::MultiPart(vec![text, binary]);
```

## Session Management

```rust
// Start a multi-agent conversation
let mut session = A2ASession::new(
    "room-1".into(),
    vec!["orchestrator".into(), "translator".into()],
);

session.add_message(msg);
session.add_message(response);

// Check history
for msg in session.history() {
    println!("{} → {}: {:?}", msg.from, msg.to, msg.message_type);
}

session.close();
```

## Installation

```toml
[dependencies]
agent-to-agent = "0.1"
```

## Testing

```bash
cargo test    # 54 tests covering all protocol operations
```

## License

MIT OR Apache-2.0
