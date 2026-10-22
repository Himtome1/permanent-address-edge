# permanent-address-edge

Example configs for giving game servers an address that belongs to the server, not to the machine it runs on. Then a server can move between machines and nobody has to update anything.

Full write-up: [Permanent Minecraft server addresses with a WireGuard edge and mc-router](https://packyard.ca/blog/permanent-addresses)

## The idea

Players never connect to the machine their server runs on. They connect to one small edge VPS. Each node has a WireGuard tunnel to the edge.

```
player ── TCP 25565 ──▶ edge
                        mc-router: read hostname from the handshake
                        └── WireGuard ──▶ node
                                          └── server container, bound to the node's tunnel address only
```

- **Minecraft Java:** the handshake carries the hostname the player typed. [mc-router](https://github.com/itzg/mc-router) reads it and proxies to the mapped backend. One wildcard DNS record (`*.play.example.com`) points at the edge, so there are no per-server DNS or SRV records. A hostname with no route gets its connection closed.
- **Games with no hostname in the protocol (Valheim, Terraria, Project Zomboid):** each server gets its own port on the edge, and nftables DNATs it over WireGuard to the same port on the node. Ports come from one pool for the whole fleet, so a server keeps its port when it moves.
- **Moving a server** means changing one backend at the edge. The address players saved doesn't change. Server transfers between nodes: the endpoint exists, but we haven't run one in production yet.

## Files

| File | What it shows |
|---|---|
| `examples/wg0-edge.conf` | WireGuard on the edge, one peer per node |
| `examples/wg0-node.conf` | WireGuard on a node |
| `examples/mc-router-route.md` | Adding and changing a route through mc-router's API |
| `examples/edge-nftables.conf` | DNAT + SNAT for port-based games on the edge |
| `examples/node-firewall.md` | ufw on the node, and publishing Docker ports on the tunnel address only |

Every address, key and hostname here is a placeholder (`10.0.0.x`, `example.com`, `<EDGE_PUBLIC_IP>`, `<PRIVATE_KEY>`).

## Caveats

- This doesn't make a node's IP secret. What it gives you is that nothing you hand to players contains a node address.
- The edge is one more hop and one more thing that can break. Keep it close to the nodes.
- DDoS: our edge relies on OVH standard filtering at the edge, not a game-aware filter.

This setup runs [Packyard](https://packyard.ca).

## License

MIT
