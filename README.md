# Best Networking Practices for Sonos Devices

I have followed the guidance given [here](https://help.ui.com/hc/en-us/articles/18930473041047-Best-Practices-for-Sonos-Devices) to avoid networking problems with Sonos devices.

Specifically, I have created a VLAN dedicated exclusively to Sonos devices.
Sonos devices are all wired into this VLAN.
Because they are wired, they do not communicate wirelessly (using SonosNet) which,
if not handled carefully with the right [STP](https://en.wikipedia.org/wiki/Spanning_Tree_Protocol) configuration,
can create disasterous network loops.
The embedded switches in some Sonos devices are not used.

# Sonos Controller Support

Control of Sonos devices is done using the Sonos application on a phone or Windows machine.
Since the Sonos devices have been isolated on their own separate VLAN, such communication is not possible without some remediation.

To solve some of this problem, one can turn on IGMP Snooping and Multicast DNS (mDNS) between the necessary VLANs on a Ubiquiti Dream Machine (UDM).
Unfortunately, the UDM does not also support a necessary UPnP SSDP relay capability.

Fortunately, deployment of a [multicast-relay](https://github.com/alsmith/multicast-relay) solves this problem.
It seems that this used to be deployable on a UDM in a [Docker container](https://github.com/scyto/multicast-relay).
That does not seem to be the case now.
A similar deployment might be done on a capable NAS but that would invite some exposure/risk from these VLANs.
Instead, it seems best to deploy a dedicated device for this purpose.
