# WireGuard Multi-Hop Tunnel

A secure, configurable multi-hop VPN tunnel implementation using WireGuard.

## Overview

WireGuard Multi-Hop Tunnel creates a chain of WireGuard connections to route network traffic through multiple nodes, enhancing privacy and circumventing network restrictions. This implementation uses three nodes in a chain:

1. **Input Node** (Client) - The entry point where traffic originates
2. **Forward Node** - The middle relay that forwards traffic between input and output
3. **Output Node** - The exit point that connects to the internet

The multi-hop design provides several advantages over a standard single-hop VPN:
- Enhanced privacy through multiple layers of encryption
- Better resistance against traffic correlation attacks
- Ability to bypass sophisticated network restrictions

## Architecture

```
[Client] → [Input Node] → [Forward Node] → [Output Node] → [Internet]
          172.16.0.1      172.16.0.2       172.16.0.3
```

- Each node has its own WireGuard interface (wg0)
- The nodes form a virtual private network in the 172.16.0.0/24 subnet
- Traffic is encrypted at each hop using WireGuard's cryptography

## Requirements

- Linux servers with root access for each node
- WireGuard installed on all machines
- Basic knowledge of networking and iptables

## Installation

1. Install WireGuard on each node:

```bash
# Debian/Ubuntu
sudo apt update
sudo apt install wireguard

# CentOS/RHEL
sudo yum install epel-release elrepo-release
sudo yum install kmod-wireguard wireguard-tools
```

2. Generate key pairs for each node:

```bash
# On Input Node
wg genkey | tee in_private.key | wg pubkey > in_public.key

# On Forward Node
wg genkey | tee fwd_private.key | wg pubkey > fwd_public.key

# On Output Node
wg genkey | tee out_private.key | wg pubkey > out_public.key
```

3. Transfer the public keys to their respective peers (keep private keys secure)

## Configuration

### Input Node Configuration

Create `/etc/wireguard/wg0.conf` with:

### Forward Node Configuration

Create `/etc/wireguard/wg0.conf` with:

### Output Node Configuration

Create `/etc/wireguard/wg0.conf` with:

**Note**: Replace `eth0` in the Output Node configuration with your actual internet-facing network interface.

## Usage

1. Start the WireGuard interfaces on all nodes (from output to input order):

```bash
# On Output Node
sudo wg-quick up wg0

# On Forward Node
sudo wg-quick up wg0

# On Input Node
sudo wg-quick up wg0
```

2. Configure your client to route traffic through the Input Node:

```bash
# Example: Add route on Linux client
sudo ip route add default via 172.16.0.1
```

3. Test the connection:

```bash
# Check if traffic is routed through the tunnel
curl ifconfig.me
```

4. To stop the tunnel, bring down interfaces (from input to output order):

```bash
# On Input Node
sudo wg-quick down wg0

# On Forward Node
sudo wg-quick down wg0

# On Output Node
sudo wg-quick down wg0
```

## Security Considerations

- Each node should be on a different provider/network for optimal security
- Do not run all nodes on the same physical machine or same hosting provider
- Regularly rotate keys for enhanced security
- Consider using DNS over HTTPS or DNS over TLS to prevent DNS leaks
- Use a dedicated user account with limited privileges for running WireGuard

## Troubleshooting

### Connection Issues

1. Check if WireGuard interfaces are up on all nodes:
```bash
sudo wg show
```

2. Verify firewall rules allow WireGuard ports:
```bash
sudo iptables -L
```

3. Check routing tables:
```bash
ip route show
ip rule show
```

4. Examine WireGuard logs:
```bash
journalctl -xeu wg-quick@wg0
```

### Packet Loss or Latency

- Increase MTU settings if needed:
```bash
# Add to [Interface] section of wg0.conf
MTU = 1420
```

- Check network conditions between nodes

## Advanced Configuration

### Adding More Hops

To add additional hops, follow a similar pattern with unique IP addresses for each new node. Modify routing tables and peer configurations accordingly.

### Split Tunneling

To route only specific traffic through the tunnel:
1. Modify AllowedIPs on the Input Node to include only specific subnets
2. Configure local routing rules on the client

## License

This project is licensed under the MIT License - see the LICENSE file for details.