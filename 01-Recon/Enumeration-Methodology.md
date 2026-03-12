
# Enumeration Methodology

A framework for structuring enumeration across external and internal penetration tests. Not a step-by-step guide — a systematic approach that accommodates dynamic environments while ensuring nothing is skipped.

---

## Three Levels

Enumeration is divided across three levels, each building on the last:

| Level | Scope |
|-------|-------|
| Infrastructure-based | Networks, domains, gateways, cloud — the outer perimeter |
| Host-based | Services, ports, interfaces running on a specific target |
| OS-based | Internal OS config, users, processes, files — requires access |

---

## The 6 Layers

Think of each layer as a wall. The goal is to find the gap — not blow through it blindly. Some gaps look promising but lead nowhere. Identify all of them before committing.

| # | Layer | Goal | Key Categories |
|---|-------|------|----------------|
| 1 | **Internet Presence** | Find everything the target exposes externally | Domains, subdomains, vHosts, ASN, netblocks, IPs, cloud instances, WAF/CDN |
| 2 | **Gateway** | Understand how the perimeter is protected | Firewalls, DMZ, IPS/IDS, EDR, proxies, NAC, VPN, Cloudflare, network segmentation |
| 3 | **Accessible Services** | Map every service reachable from outside or inside | Service type, version, port, config, interface, functionality |
| 4 | **Processes** | Understand what each service is doing with data | PIDs, processed data, tasks, sources, destinations |
| 5 | **Privileges** | Find misconfigurations in permissions | Users, groups, permissions, restrictions, environment |
| 6 | **OS Setup** | Understand how the system is configured internally | OS type, patch level, network config, sensitive files, config files |

> [!info] Layers 1 & 2 (Internet Presence, Gateway)
> These don't directly apply to internal-only infrastructure like Active Directory. They're primarily relevant for external black-box assessments. AD and intranet enumeration are covered in later modules.

> [!tip] This Module — Layer 3
> The Footprinting module focuses almost entirely on **Layer 3: Accessible Services** — understanding what's running, how it works, and how to interact with it.

---

## Key Principles

**Methodology ≠ checklist.** The tools and commands change. The systematic approach doesn't. A cheat sheet of commands is an aid, not the methodology itself.

**Time is always limited.** A four-week test won't find everything. Someone studying a target for months will always know more. The methodology exists to maximise coverage within the window you have.

**Not every gap leads further.** Finding a vulnerability doesn't mean it's the right path. Enumerate broadly first, then prioritise.

**There is almost always a way in.** If the engagement ends without a finding, that means the way in wasn't found — not that it doesn't exist.
