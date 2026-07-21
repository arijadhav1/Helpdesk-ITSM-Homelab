# Physical Network Setup

Before touching Proxmox or pfSense, I had to solve a basic problem. My AT&T gateway lives downstairs in the TV room. The OptiPlex needed to live somewhere else. This is how I got a working wired connection between them.

## The Setup

I had a Nighthawk mesh system sitting unused, with a satellite node already placed in my room. The plan was simple. Connect the mesh's main router to the AT&T gateway, then plug the OptiPlex straight into the satellite node with an Ethernet cable. No Wi-Fi involved for the server itself.

## First Attempt, IP Passthrough

My first instinct was to set up IP Passthrough on the AT&T gateway, so the Nighthawk would get the real public IP and act as the main router. I found the setting in the AT&T gateway's Advanced menu, then talked myself out of using it. pfSense only needs to route my one server, not my whole house. A normal private IP handed down from AT&T works fine. Skipped passthrough and saved myself a config change I didn't need.

## Wrong Port

Plugged the cable from the AT&T gateway into what I thought was the Nighthawk's WAN port. Wrong port. The Nighthawk has two ports next to each other, one labeled "Ethernet" and one labeled "Internet." I had it in Ethernet. Moved it to Internet.

## Cable Diagnostic Came Back Bad

Even after fixing the port, the Nighthawk still showed no internet. Ran a cable diagnostic on the AT&T side. Internet Access came back red, but downstream and upstream were both green. The physical signal was fine, something else was broken. Power cycled both devices, AT&T gateway first, then the Nighthawk. No change.

## The Real Problem

I have two separate Netgear routers, the mesh system and an old standalone Nighthawk I wasn't even planning to use. The mesh router's WAN port was connected to both the AT&T gateway and the standalone router at the same time. Two upstream connections fighting each other is what was actually breaking things, not the cabling or the AT&T side at all.

Unplugged the standalone router from the mesh. Internet came up immediately.

## Conclusion

This took longer than it should have, mostly because I was troubleshooting the wrong layer at almost every step. I looked into IP Passthrough before realizing I didn't need it. I had the cable in the wrong port on the Nighthawk. I mixed up which physical device was actually the mesh router versus an old standalone Nighthawk I wasn't using. I ran cable diagnostics on the AT&T gateway that came back clean, because the AT&T side was never actually broken. The real issue was a second router quietly plugged into the mesh at the same time as AT&T, feeding it a conflicting connection. Once I found that and unplugged it, everything worked.

Nothing here was complicated once I knew what was wrong. Getting there just meant ruling out one layer at a time, starting with the wrong assumptions first.
