# netactuate-ansible-anycast-dns

Production-grade anycast DNS deployment using KnotDNS across NetActuate's global PoP
network. Based on a real deployment serving 30+ locations. Use it as a reference
architecture for anycast DNS, authoritative DNS at the edge, or AS112 blackhole DNS.

## What This Deploys

For each node in your inventory:

1. NetActuate compute VM running Ubuntu 24.04 LTS
2. BIRD2 BGP sessions announcing your DNS anycast prefix
3. KnotDNS authoritative server listening on the anycast IP
4. Zone data deployed and loaded across all nodes
5. Per-node identification via `hostname.as112.arpa` TXT records

End state: DNS queries to your anycast IP are answered by the nearest PoP.

## What Is AS112

[AS112](https://www.as112.net/) is a community-operated anycast DNS service that sinks
reverse DNS queries for RFC 1918 private address space (10.0.0.0/8, 172.16.0.0/12,
192.168.0.0/16) that should never leak to the public Internet.

The included zone data implements AS112 as a working reference. **You can replace these
zones with your own** — the infrastructure (BGP + KnotDNS + anycast) works the same
regardless of what zones you serve.

## Prerequisites

### NetActuate Account Requirements

| Requirement | Where to Get It |
|-------------|----------------|
| API key | [portal.netactuate.com/account/api](https://portal.netactuate.com/account/api) |
| BGP group ID | Contact support@netactuate.com |
| Contract ID | Portal API page |
| ASN | Provided with BGP group |
| IPv4/IPv6 prefix | Assigned with BGP group |

### Control Node Setup

#### macOS

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
pip install ansible
pip install git+https://github.com/netactuate/naapi.git@vapi2
ansible-galaxy collection install git+https://github.com/netactuate/ansible-collection-compute.git,vapi2
```

#### Linux

```bash
sudo apt install python3-venv python3-pip
python3 -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
pip install ansible
pip install git+https://github.com/netactuate/naapi.git@vapi2
ansible-galaxy collection install git+https://github.com/netactuate/ansible-collection-compute.git,vapi2
```

#### Windows (WSL2)

Install WSL2 with Ubuntu 24.04, then follow the Linux instructions.

#### AI-Assisted (Claude Code / Cursor / Copilot)

```
Deploy anycast DNS on NetActuate:

- API Key: <KEY>
- Contract ID: <ID>
- BGP Group ID: <ID>
- ASN: <ASN>
- IPv4 Prefix: <PREFIX>/24 (DNS IP: <IP>)
- Locations: LAX, FRA, SIN, LHR, TYO (5 PoPs)
- Plan: VR4x2x50

Please run all 3 playbooks and show me how to verify DNS is working.
```

## Configuration

### group_vars/all

| Variable | Type | Description | Example |
|----------|------|-------------|---------|
| `auth_token` | string | NetActuate API key | `"abc123..."` |
| `bgp_group` | string | BGP group ID | `"12345"` |
| `contract_id` | string | Billing contract ID | `"67890"` |
| `bgp_networks` | dict | IPv4/IPv6 prefixes | See all.example |
| `knot_listen_ipv4` | string | Anycast IPv4 for KnotDNS | `"192.0.2.1"` |
| `knot_listen_ipv6` | string | Anycast IPv6 for KnotDNS | `"2001:db8::1"` |
| `knot_operator_email` | string | SOA contact email | `"admin@example.com"` |

### Substituting Your Own Zones

1. Replace zone files in `roles/knot/files/` with your own
2. Edit the `zone:` section in `roles/knot/templates/knot.conf.j2`
3. Update listen addresses in `group_vars/all` to match your anycast IPs

## Deployment Order

```bash
source .venv/bin/activate

# Step 1: Provision nodes
ansible-playbook createnode.yaml

# Step 2: Configure BGP
ansible-playbook bgp.yaml

# Step 3: Install and configure KnotDNS
ansible-playbook knotinstall.yaml
```

## Validation

```bash
# Check KnotDNS is running
ssh ubuntu@<node-ip> sudo knotc status

# Query the anycast IP
dig @YOUR_ANYCAST_IP hostname.as112.arpa TXT +short

# Expected: TXT records identifying the node that answered
# "Anycast DNS hosted by NetActuate"
# "Node: dns-LAX.example.com"

# Check zone status
ssh ubuntu@<node-ip> sudo knotc zone-status

# Verify from multiple locations using ping.pe or similar
```

## Teardown

```bash
ansible-playbook deletenode.yaml
```

## Related Resources

- [AS112 Project](https://www.as112.net/)
- [KnotDNS Documentation](https://www.knot-dns.cz/docs/)
- [netactuate-ansible-bgp-bird2](https://github.com/netactuate/netactuate-ansible-bgp-bird2) — BGP foundation
- [NetActuate Portal](https://portal.netactuate.com)

## Need Help?

- NetActuate support: support@netactuate.com
