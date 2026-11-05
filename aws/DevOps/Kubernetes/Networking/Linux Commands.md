[[DevOps/Kubernetes/Networking/Networking|Networking]]

https://notes.kodekloud.com/docs/Certified-Kubernetes-Administrator-CKA/Networking/Prerequisite-Switching-Routing-Gateways-CNI-in-kubernetes/page
Quick reference — useful commands

| Command                                               | Description                                                                |
| ----------------------------------------------------- | -------------------------------------------------------------------------- |
| `ip link`                                             | List and modify network interfaces on the host.                            |
| `ip addr`                                             | View IP addresses assigned to interfaces.                                  |
| `ip addr add` `<addr>/<prefix>` `dev` `<iface>`       | Assign an IP address to an interface (wrap `<...>` in backticks as shown). |
| `ip route` or `route -n`                              | View the kernel routing table.                                             |
| `ip route add` `<network>/<prefix>` `via` `<gateway>` | Add a route to the routing table.                                          |
| `ip route add default via` `<gateway>`                | Add a default route (0.0.0.0/0).                                           |
| `cat /proc/sys/net/ipv4/ip_forward`                   | Check IP forwarding state (0 = disabled, 1 = enabled).                     |
| `sudo sysctl -w net.ipv4.ip_forward=1`                | Enable IP forwarding temporarily.                                          |
| Edit `/etc/sysctl.conf` and run `sudo sysctl -p`      | Make IP forwarding persistent.                                             |
