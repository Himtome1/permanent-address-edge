# mc-router routes (Minecraft Java)

[mc-router](https://github.com/itzg/mc-router) listens on TCP 25565 on the edge, reads the hostname from the client's handshake, and proxies to the backend mapped to that name. A hostname with no route gets its connection closed.

DNS: one wildcard record, no per-server records.

```
*.play.example.com.  60  IN  A  <EDGE_PUBLIC_IP>
```

Run mc-router with its HTTP API bound to the edge's localhost only:

```
mc-router -port 25565 -api-binding 127.0.0.1:25581
```

Add a route (backend = the node's tunnel address and the server's port):

```
curl -X POST http://127.0.0.1:25581/routes \
  -H 'Content-Type: application/json' \
  -d '{"serverAddress": "yourserver.play.example.com", "backend": "10.0.0.2:25565"}'
```

Move the server to another node: point the same name at the new backend. Players keep the address they saved.

```
curl -X DELETE http://127.0.0.1:25581/routes/yourserver.play.example.com
curl -X POST http://127.0.0.1:25581/routes \
  -H 'Content-Type: application/json' \
  -d '{"serverAddress": "yourserver.play.example.com", "backend": "10.0.0.3:25565"}'
```

Note: routes added through the API are not kept across an mc-router restart in our setup. Treat the router as a cache and have whatever owns the mapping re-push every route on a schedule (we do it every two minutes).
