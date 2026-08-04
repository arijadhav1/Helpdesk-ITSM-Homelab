# Active Directory Domain Controller Setup
 
Now we are going to create our Active Directory VM in our Proxmox environment. This is going to be the domain controller for the whole Meridian Logistics lab, everything else, osTicket, Zabbix, client machines, all authenticates against this box.
 
## Downloading Windows Server 2022
 
First thing, downloaded the Windows Server 2022 ISO from the Microsoft website.
 
<img width="727" height="312" alt="Please select your Windows Server 2022 download" src="https://github.com/user-attachments/assets/10a69554-641c-4f6c-9505-df9d1a73c34f" />

## Uploading the ISO into Proxmox
 
Same process as pfSense. Click `ari`, local storage, ISO Images, Upload, and upload the ISO.
 
<img width="1507" height="663" alt="Screenshot 2026-08-03 at 11 36 49 AM" src="https://github.com/user-attachments/assets/a9ccbe3b-d0c1-40c9-b6c3-71a22cb429e4" />


Hit upload. Once uploaded, pressed Create VM at top right.
 
## Creating the VM
 
In the General tab, named the VM `ad-dc01`.
 
Went to the OS tab. Selected the ISO image, type as Microsoft Windows, version as 11/2022.
 
<img width="759" height="569" alt="• THe CD OVD dặc mage fin (so!" src="https://github.com/user-attachments/assets/afd46689-b9e2-40f2-b192-0b2ce1329b44" />

Left the System tab as is but checked the Qemu agent option.
 
In the Disks tab, set the size to 60GB and storage to `local-lvm`.
 
<img width="757" height="569" alt="Create Virtual Machine" src="https://github.com/user-attachments/assets/96da5793-c045-4c05-8746-7b2eaba4740f" />

Set the cores to 4.
 
<img width="757" height="570" alt="Create Virtual Maching" src="https://github.com/user-attachments/assets/1f68dadd-aae1-4d7a-b0bd-374b363a405d" />


Set memory to 8192 (8 GB).
 
<img width="758" height="152" alt="Create Virtual Machine" src="https://github.com/user-attachments/assets/eb1190a8-db2b-4147-b5d8-68113cb16084" />

For the Network tab, set the bridge to `vmbr1` and the VLAN tag to `10`.
 
Here's why. `vmbr1` is the bridge I built during pfSense setup, it's what carries traffic to the lab side of the network instead of my home network. VLAN tag `10` is the Servers VLAN I created on pfSense, `10.10.20.0/24`. Tagging the VM's NIC with `10` puts it on that VLAN specifically, not the Clients VLAN or the old flat LAN. The AD DC is a server, so it belongs on the Servers segment with the rest of the infrastructure I'm building, osTicket and Zabbix will get the same treatment when I create them. Keeping servers and clients on separate VLANs is the whole point of segmenting the network in the first place, so a compromised client machine can't reach the domain controller directly.
 
<img width="761" height="232" alt="Create Virtual Machine" src="https://github.com/user-attachments/assets/d15ed7c9-9de2-4f61-bcf7-5a53875bb2fa" />

Hit finish.
 
## Booting and Installing Windows Server
 
Started the VM and opened the console. It said no bootable option, so I pressed a random key and ended up on the boot device selection screen.
 

<img width="368" height="287" alt="Please select boot device" src="https://github.com/user-attachments/assets/a1525ddf-6e82-4568-a332-e412afb76e78" />

Selected the second option, DVD-ROM, where the ISO is attached.
 
Left language, time, and keyboard screen as default and pressed Install now.
 
For operating system, selected Windows Server 2022 Standard Evaluation (Desktop Experience).
 
<img width="714" height="555" alt="Microsoft Server Operating System Setup" src="https://github.com/user-attachments/assets/b47c64e4-1d99-47f5-82fd-a7266e2dcf5d" />

Hit next, accepted license terms, hit next again.
 
For installation type, selected the second option since this is a fresh install on a blank virtual disk.
 
<img width="694" height="531" alt="• Microsoft Server Operating System Setup" src="https://github.com/user-attachments/assets/f35ab5b5-afda-4577-b93a-cca1b9222de0" />

Selected Drive 0 as the disk.
 
<img width="642" height="484" alt="   Microsoft Server Operating System Setup" src="https://github.com/user-attachments/assets/f9ce5d70-7bbb-4010-929f-706dc70fd244" />

Now installing.
 
In the customize settings screen:
 
- Username: `Administrator`
- Password: set to a strong local admin password
Signed in. Now on the Windows Server desktop.
 
<img width="1281" height="808" alt="WELCOME SO SERVER MANAGER" src="https://github.com/user-attachments/assets/a81461e3-747e-45f9-a0e3-2d7d99a0a898" />

## Setting a Static IP
 
Now setting a static IP for this VM. Opened network and internet settings, adapter settings, and went into the ethernet adapter properties.
 
Set:
 
- IP address: `10.10.20.10` (within our Servers range)
- Subnet mask: `255.255.255.0`
- Gateway: `10.10.20.1` (pfSense's Servers interface)
- DNS server: `127.0.0.1`
Pointing DNS at itself makes sense here since this VM is about to become the DNS server for the domain once AD DS is installed.
 
<img width="1269" height="797" alt="Screenshot 2026-08-03 at 12 29 53 PM" src="https://github.com/user-attachments/assets/024cdc09-4d44-456e-9682-277fc2ce47b9" />

## Renaming the Server
 
Went into System Properties. To do so, searched This PC, went to Properties, and changed the name to `ARI-ADDC01`.
 
Reboot and restart.
 
## Installing Active Directory Domain Services
 
After reboot, in Server Manager, went to Add Roles and Features, role-based install, selected this server, checked Active Directory Domain Services, and installed.
 
<img width="788" height="567" alt="Bo Add Roles and Festures Wicard" src="https://github.com/user-attachments/assets/7d0316c5-31e7-4673-9dd0-c98e9baf2050" />

## Promoting to a Domain Controller
 
After installation, needed to promote the server to a domain controller. On the top right of Server Manager, clicked the flag and clicked "Promote this server to a domain controller."
 
<img width="599" height="338" alt="Manage Tools View Help" src="https://github.com/user-attachments/assets/061559e6-a5b3-4c03-ac8d-9f4953ab1d03" />

Now in Deployment Configuration. For deployment operation, clicked "Add a new forest." This is the top-level container in AD, it'll be the backbone for the rest of the homelab.
 
Named the root domain `ari.local`.
 
<img width="763" height="566" alt="Es Active Directoey Domain Services Configuration Wizard" src="https://github.com/user-attachments/assets/5f5c64a3-372f-4915-8048-e69882e713f2" />

Left functional levels at default. Set the DSRM password. If AD ever gets corrupted and needs repair, this password is what gets used for recovery.
 
DSRM Password: set to a strong recovery password, kept separate from the domain admin password
 
<img width="763" height="565" alt="Ra Active Directory Demain Services Configuration Wizard" src="https://github.com/user-attachments/assets/96ede9b6-451c-4861-841a-77baf6c7f0cb" />

Next was DNS options. Don't need to create DNS delegation for this lab, so skipped and went next.
 
Additional options next, left the NetBIOS name as `ARI`, next.
 
Paths, left as default, next.
 
Reviewed options once more.
 
Prerequisites check, validating the machine is ready for promotion.
 
<img width="762" height="561" alt="Pa Active Directory Demain Services Configurabon Wizard" src="https://github.com/user-attachments/assets/e006aef9-a4d3-41e5-a0fe-15e8d6cd570e" />

Hit install. After reboot, AD should be running.
 
<img width="799" height="488" alt="RAAdministrator" src="https://github.com/user-attachments/assets/643881d7-d2b3-4152-8896-d3a543f90860" />

## Building the OU Structure
 
Now creating the OU structure. An OU (Organizational Unit) is a folder for organizing accounts and machines logically, it's the level at which Group Policies get applied.
 
Went back to Server Manager. Tools, Active Directory Users and Computers. Opens up a folder-like section.
 
Right-clicked `ari.local`, New, Organizational Unit. Creating three to start: Employees, IT, and Servers. Unchecked "Protect container from accidental deletion" on each so I can delete or restructure them later without a fight.
 
Should now have 3 new OUs under `ari.local`.
 
<img width="755" height="525" alt="Active Directory Users and Computers" src="https://github.com/user-attachments/assets/80da6ad6-6bc0-4be4-9038-8467dc2ebfc0" />

## Creating Test Users
 
Now creating the first test user inside the Employees OU.
 
Right-clicked Employees, New, User.
 
- First name: David
- Last name: Webb
- User logon name: `dwebb@ari.local`
- Password: set to a test password meeting the domain policy
Unchecked "user must change password at next logon" and checked "password never expires," just for testing purposes.
 
<img width="434" height="379" alt="New Object - User" src="https://github.com/user-attachments/assets/4edc0d51-e081-4122-865e-0bd6cd88cbb9" />

<img width="436" height="376" alt="New Object - User" src="https://github.com/user-attachments/assets/08818385-65a9-49c8-b524-abe15d3e13b3" />

Did the same three more times:
 
- First name: Sharon, Last name: Kelly, logon: `skelly@ari.local`, password: set to a test password meeting the domain policy
- First name: Mike, Last name: Scott, logon: `mscott@ari.local`, password: set to a test password meeting the domain policy
- First name: Adam, Last name: Cooper, logon: `acooper@ari.local`, password: set to a test password meeting the domain policy
Now have 4 users in Employees.
 
<img width="755" height="532" alt="¢ 0 2 000000н8009" src="https://github.com/user-attachments/assets/f34feed7-2b8a-4979-927e-10d175e0fb14" />

## Creating a Security Group
 
Next, creating a security group. The idea is to apply permissions and policies to the group instead of to each user one by one, everyone in the group inherits whatever gets applied to it.
 
Right-clicked Employees, New, Group. Named it `All-Employees`, group scope Global, group type Security.
 
<img width="436" height="374" alt="New Object - Group" src="https://github.com/user-attachments/assets/6db1c920-e2cd-4592-9af6-7b68f6d292d5" />

Added all four users to the group. Double-clicked the group, Properties, Members, Add, typed everyone's username, OK, OK.
 
<img width="403" height="460" alt="All-Employees Properties" src="https://github.com/user-attachments/assets/4b56f69a-82b3-4725-b0cb-5fc6362e41bf" />

## Setting a Password Policy via Group Policy
 
After adding them, setting a password policy via Group Policy.
 
In Server Manager, Tools, Group Policy Management. Expanded `ari.local` on the left, expanded Domains, expanded `ari.local` once more. Right-clicked Default Domain Policy, Edit.
 
<img width="1198" height="566" alt="Screenshot 2026-08-04 at 12 07 35 PM" src="https://github.com/user-attachments/assets/d395886e-ef21-433a-a961-7cdeb4b0f641" />

Navigated to password policy settings: Computer Configuration, Policies, Windows Settings, Security Settings, Account Policies, Password Policy.
 
Configured:
 
- Minimum password length: 8
- Password must meet complexity requirements: Enabled
- Maximum password age: 90 days
- Enforce password history: 5
<img width="785" height="570" alt="I Name Resettien Polay" src="https://github.com/user-attachments/assets/08d6cec7-c023-49cc-8cf8-3027b56fcf45" />

## Creating the IT Staff Group
 
Also creating a group for IT staff, for later use when handling tickets. Using David Webb for IT.
 
Right-clicked David Webb, Move, selected the IT OU.
 
Created a new `IT-Staff` group, group scope Global, group type Security. Same steps as before to add David Webb as a member.
 
<img width="994" height="530" alt="тот, ста" src="https://github.com/user-attachments/assets/0c30bf91-c137-4a07-b5e6-b5fe3d73dcb9" />

## Result
 
```
Domain:       ari.local
DC Hostname:  ARI-ADDC01
DC IP:        10.10.20.10/24 (Servers VLAN)
OUs:          Employees, IT, Servers
Groups:       All-Employees (4 members), IT-Staff (David Webb)
Password policy: min length 8, complexity enabled, 90-day max age, history of 5
```
 
AD DC is up, domain is live, OU structure and groups are in place, and password policy is enforced at the domain level. Next up is osTicket on the Servers VLAN, then Zabbix.
