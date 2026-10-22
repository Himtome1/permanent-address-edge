# Node firewall and Docker port publishing

Docker's published ports skip ufw. If a container port is published on all interfaces, the firewall doesn't save you. So publish game ports on the node's WireGuard address only:

```
# docker run: bind to the tunnel address, not 0.0.0.0
docker run -d -p 10.0.0.2:25565:25565 <IMAGE>
```

```yaml
# docker compose
services:
  game:
    image: <IMAGE>
    ports:
      - "10.0.0.2:25565:25565"
```

Then the node's firewall takes SSH from a short allowlist and nothing else:

```
ufw default deny incoming
ufw default allow outgoing
# the node dials out to the edge, so WireGuard needs no inbound rule
ufw allow from <ADMIN_IP> to any port 22 proto tcp
ufw enable
```

Check from outside that nothing else answers on the node's public address:

```
nmap -Pn -p- <NODE_PUBLIC_IP>
```
