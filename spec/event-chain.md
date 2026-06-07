# Chained Event Protocol (CEP) Specification v0.1

## Event Structure

```
Event {
    id: UUID v4
    type: "task.submit" | "task.complete" | "skill.call" | 
          "knowledge.learn" | "device.register" | "device.revoke" |
          "core.law.update" | "sync.checkpoint"
    payload: JSON (any valid JSON object)
    prev_hash: SHA-256 hex (hash of previous event on THIS device)
    root_hash: SHA-256 hex (hash of the last globally synchronized event)
    signature: Ed25519 hex (signed by device key)
    device_id: Ed25519 public key hex (device's identity)
    timestamp: ISO 8601 UTC
}
```

## Hash Computation

```
hash = SHA-256(
    id + "|" +
    type + "|" +
    JSON(payload) + "|" +
    prev_hash + "|" +
    root_hash + "|" +
    device_id + "|" +
    timestamp + "|" +
    signature
)
```

## Chain Invariants

1. **prev_hash** must equal SHA-256(previous_event) on the same device
2. **root_hash** must equal the hash of the last globally synced event
3. **signature** must be valid for the event content under device_id's public key
4. **device_id** must be a valid sub-key under the master public key
5. Events are immutable — changing any field invalidates all subsequent hashes

## Master Key Hierarchy

```
Master Key (Ed25519)
├── Grants: register devices, update core laws, revoke devices
├── Signs: device registration certificates
└── Device Key (Ed25519)
    ├── Grants: sign events for ONE device
    ├── Signs: all events on that device
    └── Revocation: master key signs "revoke {device_id}" event
```
