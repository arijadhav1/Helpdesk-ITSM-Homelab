# Proxmox Installation and pfSense Setup

This covers turning the OptiPlex into a Proxmox host and standing up pfSense as the first VM on it. This is the foundation the rest of Meridian Logistics gets built on top of.

## Installing Proxmox

Downloaded the Proxmox VE ISO and put it on a USB flash drive using Rufus. I originally tried doing this on my Mac, then remembered Rufus is Windows-only and switched to a Windows machine partway through. Should've just started there.

<img width="487" height="644" alt="9 Rufus 4 15 2396" src="https://github.com/user-attachments/assets/0f681725-0c65-4b80-86c0-8c84f568f152" />


Booted the OptiPlex from the USB, hit F12 to bring up the boot menu, and selected the drive.

<img width="1028" height="652" alt="Screenshot 2026-07-22 at 4 31 15 PM" src="https://github.com/user-attachments/assets/20443fc6-3d5a-47af-bcac-f8f75aeb6c5d" />


Ran through the Proxmox installer. When it came time to pick a target disk, my SSD wasn't showing up at all, only the HDD. Rebooted into BIOS, went to SATA Operations, and found the controller was set to RAID instead of AHCI. Switched it to AHCI since I'm only running one drive, no need for RAID's multi-disk logic. Booted back into the installer and the SSD showed up immediately.


<img width="863" height="514" alt="Screenshot 2026-07-22 at 4 31 47 PM" src="https://github.com/user-attachments/assets/e7bed923-9367-4dbf-8867-2b6f390fb42e" />

<img width="1046" height="637" alt="Screenshot 2026-07-22 at 4 32 09 PM" src="https://github.com/user-attachments/assets/bbef1c96-288e-4e3c-8564-645ae0dc00db" />


Selected the SSD, went through location and timezone, then set the root password and an admin email.

Next was Management Network Configuration, setting the IP address, gateway, and DNS server:

- Hostname: `ari.homelab.local`
- IP address: `10.0.0.50`
- Gateway: `10.0.0.1`
- DNS server: `10.0.0.1`

Went with a static IP on my home network's private range rather than a public-facing address, so Proxmox is only reachable from inside my own network, never from the internet directly.

<img width="1038" height="633" alt="Screenshot 2026-07-22 at 4 32 49 PM" src="https://github.com/user-attachments/assets/78972999-d02c-4e4b-b1c1-1513bcd37e9a" />


Hit Install. It rebooted on its own and came up running Proxmox.

## Accessing the Web UI

Connected my MacBook to the same network as the OptiPlex and went to `https://10.0.0.50:8006`. Clicked through the self-signed cert warning and landed on the login screen. Logged in with `root` and the password I set during install.

<img width="1512" height="972" alt="PROXMOX WAIL" src="https://github.com/user-attachments/assets/f77d2c51-f753-4bc1-862f-b23d80eb01bf" />


There was an error at the bottom of the dashboard. Proxmox defaults to the enterprise repository, which needs a paid subscription I don't have. Fixed it by going to node `ari` → Updates → Repositories, disabling both enterprise repos, then clicking Add and selecting the no-subscription option. Reloaded and it updated cleanly after that.

<img width="1202" height="618" alt="Screenshot 2026-07-22 at 3 15 52 PM" src="https://github.com/user-attachments/assets/1c58494d-0278-46e2-9f91-6dcdd2f43596" />


## Setting Up pfSense

Downloaded the pfSense CE (Community Edition) installer, AMD64 build, from Netgate's site. Uploaded it into Proxmox under node `ari` → local storage → ISO Images → Upload.

<img width="1147" height="591" alt="Screenshot 2026-07-22 at 3 24 56 PM" src="https://github.com/user-attachments/assets/3b41c8a2-9413-4a7f-964b-c275881704f8" />


Clicked Create VM, named it `pfsense`, selected the ISO I'd just uploaded. Kept the System tab defaults. Set the disk to 20GB, plenty for pfSense. Bumped CPU to 2 cores. Left memory at the default 2048 MiB. For network, only `vmbr0` existed at this point, so I left it on that for now and finished creating the VM.

<img width="762" height="571" alt="Creste Vetust Machine" src="https://github.com/user-attachments/assets/6adbd677-7d6c-424a-b934-937da7b6a139" />


### Building the second network

Before going any further, I needed a second bridge. Here's why: pfSense's whole job is sitting between two networks and controlling what passes between them, my home network on one side, and the isolated lab network (AD server, ticketing, monitoring) on the other. With only one bridge existing, there was no lab network yet for pfSense to actually manage. The second bridge is what brings that network into existence in the first place.

Went to node `ari` → System → Network → Create → Linux Bridge. Named it `vmbr1`, set the bridge port to `nic0` (the physical NIC not already in use by `vmbr0`), left a comment noting it's the lab LAN. Created it and hit Apply Configuration.

<img width="1147" height="607" alt="XPROXMOX Vinual Envionmert 92" src="https://github.com/user-attachments/assets/b84cec4b-3121-42eb-8492-7d801489639e" />


Went back into the pfSense VM's Hardware tab, clicked Add → Network Device, and set the bridge to `vmbr1`. That gives pfSense its second interface.

<img width="1149" height="587" alt="XPROXMOX Vital Enviroomert 9 23" src="https://github.com/user-attachments/assets/4cb30a7e-20c9-4af1-a9e1-084ba6c46214" />


### Installing pfSense

Started the VM and opened the console. Ran through the Netgate installer.

<img width="716" height="437" alt=" Not Secure - 10 0 0 50" src="https://github.com/user-attachments/assets/11dd96bc-a83b-4dd9-8882-19b63f04b1ee" />


Assigned interfaces: `em0` as WAN, `em1` as LAN, matching `vmbr0` and `vmbr1` respectively.

<img width="717" height="436" alt="Netgate Installer - vl Z-RELEASE" src="https://github.com/user-attachments/assets/00511aa0-78bd-4716-8a52-c75c667afb33" />

<img width="719" height="433" alt="X Not Secure - 10 0 0 50" src="https://github.com/user-attachments/assets/2f6898f9-9faf-455d-ae7b-8c82b3b4c13c" />


Left WAN on DHCP so it pulls an address from my home network automatically. For LAN, changed it to a static configuration:

- IP address: `10.10.10.1/24`
- DHCP range start: `10.10.10.100`
- DHCP range end: `10.10.10.199`

Left `.2` through `.99` open for devices I want to give fixed IPs later, like the AD server, osTicket, and Zabbix. `.100` through `.199` is the dynamic pool for anything that just needs an address without me caring which one. Splitting it this way avoids any collision between static and dynamic assignments down the line.

<img width="722" height="433" alt="•••" src="https://github.com/user-attachments/assets/60d5dde7-4fb5-4729-a646-642fb2e7656a" />


Confirmed the interface assignment, hit Continue. Got prompted about pfSense Plus, a paid subscription tier, and chose Install CE instead since that's the free version and does everything I need.

<img width="722" height="430" alt=" Not Secure - 1000 50" src="https://github.com/user-attachments/assets/3708dfea-ecea-4b00-a83a-804e6cc5b631" />


Went with the default ZFS file system and GPT partition scheme, stripe as the ZFS layout since I'm only using a single disk. Confirmed the disk, confirmed the wipe, and installed the current stable version, 2.8.1.

<img width="720" height="435" alt="X Not Secure - 10 00 50" src="https://github.com/user-attachments/assets/23b5bc88-d13a-4bcb-8382-f9a78eb3dfd3" />

<img width="721" height="431" alt="Netgate Installer - vl 2-RELEASE" src="https://github.com/user-attachments/assets/5fa9b9cf-5a6b-4bd6-9100-2760c2c20220" />

<img width="722" height="425" alt="X Not Sacure - 1000 60" src="https://github.com/user-attachments/assets/fbf666da-cd60-455d-8530-323bfc4e07c6" />

## Result

Once it rebooted, pfSense came up clean:

```
WAN (wan) -> em0 -> v4/DHCP4: 10.0.0.11/24
LAN (lan) -> em1 -> v4: 10.10.10.1/24
```

<img width="724" height="434" alt=" Not Socure -10 00 50" src="https://github.com/user-attachments/assets/ff292eb3-bff2-402f-a038-a2e53322de6a" />


WAN pulled an address from my home network as expected, and LAN is live on `10.10.10.1`, the gateway for the lab network going forward. Proxmox is installed, reachable, and updating properly. pfSense is installed with both interfaces correctly assigned. This is the foundation everything else gets built on top of, next is VLAN design and the actual firewall rules, then the AD DC, osTicket, and Zabbix VMs.
