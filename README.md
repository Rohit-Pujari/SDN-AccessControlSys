# SDN-Based Access Control System

This project enforces host-to-host communication policy in a Mininet network using a whitelist. Only source-destination pairs present in the whitelist get OpenFlow forwarding rules; all other traffic is denied by a default drop rule.

## Submission Summary

- Problem: allow only authorized hosts to communicate inside the SDN
- Approach: whitelist-based OpenFlow rule installation on Open vSwitch
- Topology: one switch `s1` and four hosts `h1` to `h4`
- Verification: automated policy tests plus live Mininet validation

## Features

- Maintain a host whitelist in `config/whitelist.json`
- Install explicit allow rules and an implicit deny policy
- Block unauthorized host communication
- Verify live reachability against the policy
- Regression test policy consistency with `unittest`

## Project Layout

- `config/whitelist.json`: host inventory and allowed communication pairs
- `sdn_acl/policy.py`: validates policy, builds connectivity matrix, generates flow rules
- `sdn_acl/runtime.py`: installs and inspects Open vSwitch flow rules
- `topology.py`: starts the Mininet topology and applies ACL rules
- `verify_access.py`: checks switch rules and runs connectivity verification
- `tests/test_policy.py`: regression tests for policy consistency

## Policy Model

Each whitelist entry is directional. If `["h1", "h2"]` is present, traffic from `h1` to `h2` is allowed. Bidirectional communication requires both `["h1", "h2"]` and `["h2", "h1"]`.

The sample configuration allows:

- `h1 <-> h2`
- `h1 <-> h3`

All communication involving `h4` is blocked.

## Quick Run

One-command demo:

```bash
chmod +x demo.sh
./demo.sh
```

Manual verification:

```bash
python3 -m unittest discover -s tests -v
sudo python3 verify_access.py
```

## Interactive Demo

Start the topology with a Mininet CLI:

```bash
sudo python3 topology.py
```

Start and exit after installing rules:

```bash
sudo python3 topology.py --no-cli
```

Verify policy enforcement:

```bash
sudo python3 verify_access.py
```

Run regression tests:

```bash
python3 -m unittest discover -s tests -v
```

## Manual Mininet Checks

Inside the Mininet CLI, use:

```bash
h1 ping -c 1 h2
h1 ping -c 1 h3
h1 ping -c 1 h4
h2 ping -c 1 h3
```

Expected behavior:

- `h1 -> h2`: allowed
- `h1 -> h3`: allowed
- `h1 -> h4`: blocked
- `h2 -> h3`: blocked

To inspect installed switch rules:

```bash
sudo ovs-ofctl -O OpenFlow13 dump-flows s1
```

## Report

Submission-ready report content is available in `report/report.md`.

## Notes

- The topology uses a single Open vSwitch bridge with OpenFlow 1.3 rules.
- Since the switch runs without a remote controller, the access-control behavior is fully driven by installed flow entries.
- ARP and IPv4 forwarding are only installed for whitelisted pairs; unmatched traffic hits the default drop rule.
