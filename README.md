# Helpdesk & ITSM Homelab

I am building a fake company's IT department from scratch, on my own hardware. Network, identity, ticketing, monitoring, all of it. The goal is to run it the way an actual helpdesk or NOC tech would: employees submit tickets, I troubleshoot and fix things, and I've got monitoring watching for problems before anyone even complains.

This is my third homelab. The first two ([AD detection & response](#) and [adversary simulation & endpoint telemetry](#)) were built around the SOC/blue team side of things. This one's different on purpose, it's built around the support side, since that's where I'm actually starting my career.

## Setup Write-ups
Every phase of this build gets documented as I go, hardware, networking, VM setup, all of it. Full write-ups live in [`/setup`](./setup):

- [Physical Network Setup](./setup/PHYSICAL-SETUP.md) — getting the OptiPlex online through my existing mesh, AT&T gateway troubleshooting
- [Proxmox + pfSense Setup](./setup/PROXMOX-PFSENSE-SETUP.md) — installing Proxmox, building the pfSense VM, setting up the WAN/LAN split

## What's in it

One PC runs Proxmox, split into a handful of VMs that make up the "company":

- **pfSense** — the router/firewall. Handles traffic between VLANs and hands out IPs
- **Windows Server 2022 AD DC** — manages who can log into what, plus groups and password policy
- **osTicket** — where the fake employees submit their problems
- **Zabbix** — watches the other VMs and tells me when something breaks

My MacBook plays the role of "the employees." It runs Windows VMs joined to the domain, so when something goes wrong, like a login failing or a machine losing network access, it's a real problem on a real machine, not something I'm just imagining.

## Hardware
Everything runs on a single dedicated box, not a laptop or cloud VM this time. I wanted the full experience of racking real (if used) enterprise gear and making it work.

- **Host**: Dell OptiPlex 7050 SFF, i7-7700, 32GB DDR4 RAM, 500GB SSD + 500GB HDD
- **Network**: TP-Link TL-SG116E, 16-port managed switch, handling VLAN tagging between pfSense and everything downstream
- **NIC**: dual-port PCIe Gigabit card, giving pfSense a proper WAN/LAN split instead of relying on one interface

The SSD holds the VMs, the HDD is backup and ISO storage. 32GB of RAM was the deciding factor when I was hunting for a host, more important than raw CPU speed for running four services at once without hitting a ceiling.

## Architecture

<img width="642" height="328" alt="PC - Proxmox VE host" src="https://github.com/user-attachments/assets/88de4c35-e8d8-4955-9047-41c6de4bf15f" />


## Stack

| Component | Tool | Role |
|---|---|---|
| Hypervisor | Proxmox VE | Hosts all server VMs on the PC |
| Network | pfSense | Routing, firewall, VLANs, DHCP |
| Identity | Windows Server 2022 | AD DC, DNS, GPOs |
| Ticketing | osTicket | Helpdesk ticket queue |
| Monitoring | Zabbix | Uptime, resource, and service alerting |
| Clients | Windows 10/11 on UTM | Domain-joined "employee" machines |

## Build phases

- [x] Source and build the host (OptiPlex 7050, 32GB RAM, dual NIC, managed switch)
- [x] Proxmox install and host config
- [ ] pfSense install, VLAN and DHCP design
- [ ] AD DC build, OUs, test users, GPOs
- [ ] osTicket install and department/category config
- [ ] Zabbix install, watching AD DC, osTicket, pfSense
- [ ] Zabbix → osTicket alert-to-ticket integration
- [ ] Client VMs on MacBook, domain-joined
- [ ] Fake ticket scenarios, worked end to end
- [ ] Documentation and screenshots

## Ticket scenarios

Once everything's up, I'll be logging real tickets here, the problem, how I tracked it down, and how I fixed it. Scenarios I'm planning to run:

1. New hire onboarding (AD account, GPO, VPN access)
2. Locked account / password reset
3. Zabbix catches a downed service and auto-opens a ticket
4. Network issue, something like a bad DHCP or DNS config
5. Remote employee can't connect over VPN
6. Same issue keeps coming up, so I write a KB article for it

## Status

Just getting started. Working on the Proxmox install right now.
