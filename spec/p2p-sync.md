# P2P Synchronization Protocol v0.1

## Message Types

| Type | Payload | Direction |
|------|---------|-----------|
| `DISCOVER` | `{network: "vita-p2p"}` | Broadcast (mDNS) |
| `HELLO` | `{root_hash, device_id, public_key, height}` | Peer → Peer |
| `SYNC_REQUEST` | `{last_known_root}` | Peer → Peer |
| `SYNC_RESPONSE` | `{events[], new_root, new_height}` | Peer → Peer |
| `EVENT_BROADCAST` | `{event}` | Broadcast (GossipSub) |
| `GOODBYE` | `{reason}` | Peer → Peer |

## Sync Algorithm

```
function sync(peer):
    1. Send HELLO with my current root hash and height
    2. Receive HELLO from peer
    3. Verify peer's device_id is in master key hierarchy
    4. Compare heights:
       - If peer.height > my.height → SYNC_REQUEST, receive events
       - If peer.height < my.height → receive SYNC_REQUEST, send events
       - If equal → done
    5. After sync, set root_hash to max(peer.root, my.root)
    6. Subscribe to EVENT_BROADCAST for real-time updates

function on_new_event(event):
    1. Verify event signature
    2. Append to local chain
    3. Broadcast via EVENT_BROADCAST to connected peers
```

## Discovery

- **Local network**: mDNS (`_vita-p2p._tcp.local`)
- **Global network**: Kad-DHT on a dedicated namespace
- **Fallback**: Manual peer address configuration
