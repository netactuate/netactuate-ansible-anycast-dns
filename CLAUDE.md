# netactuate-ansible-anycast-dns — AI Provisioning Context

Production anycast DNS using KnotDNS. Based on a real 30+ PoP deployment. BGP via BIRD2,
same as the gold standard.

## Deployment Sequence

```bash
ansible-playbook createnode.yaml    # Provision VMs
ansible-playbook bgp.yaml           # Configure BGP + BIRD2
ansible-playbook knotinstall.yaml   # Install KnotDNS + zones
```

## Required Inputs

| Input | Source | Example |
|-------|--------|---------|
| API key | portal.netactuate.com/account/api | `"abc123..."` |
| Contract ID | Portal | `"12345"` |
| BGP group ID | NetActuate support | `"67890"` |
| ASN | Provided with BGP group | `"65000"` |
| IPv4 prefix | Assigned with BGP group | `"192.0.2.0/24"` |
| Anycast DNS IP | First usable in prefix | `"192.0.2.1"` |
| Locations | Customer choice | `LAX, FRA, SIN, LHR, TYO` |

## Zone Data

The included zones implement AS112 (RFC 1918 reverse DNS sinkhole). To serve your own zones:

1. Replace `roles/knot/files/db.dd-empty` and `db.dr-empty` with your zone files
2. Edit `roles/knot/templates/knot.conf.j2` — update the `zone:` section
3. The infrastructure (BGP + anycast + KnotDNS) works identically regardless of zone content

## Validation

```bash
# Query from anywhere
dig @ANYCAST_IP hostname.as112.arpa TXT +short

# Check which node answered
dig @ANYCAST_IP hostname.bind chaos txt +short

# Verify KnotDNS status on a node
ssh ubuntu@<ip> sudo knotc status
ssh ubuntu@<ip> sudo knotc zone-status
```

## Key Files

- `knotinstall.yaml` — KnotDNS deployment playbook
- `roles/knot/templates/knot.conf.j2` — KnotDNS config (listeners, zones, logging)
- `roles/knot/files/db.dd-empty` — AS112 direct delegation zone
- `roles/knot/files/db.dr-empty` — AS112 DNAME redirection zone
- `roles/knot/templates/db.hostname.zone.j2` — Per-node identification zone
