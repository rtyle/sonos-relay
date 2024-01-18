# Best Networking Practices for Sonos Devices

I have followed the guidance given
[here](https://help.ui.com/hc/en-us/articles/18930473041047-Best-Practices-for-Sonos-Devices)
to avoid networking problems with Sonos devices.

Specifically, I have created a VLAN dedicated exclusively to Sonos devices.
Sonos devices are all wired into this VLAN.
Because they are wired, they do not communicate wirelessly (using SonosNet) which,
if not handled carefully with the right
[STP](https://en.wikipedia.org/wiki/Spanning_Tree_Protocol)
configuration, can create disasterous network loops.
The embedded switches in some Sonos devices are not used.

# Sonos Controller Support

Control of Sonos devices is done using the Sonos application on a phone or Windows machine.
Since the Sonos devices have been isolated on their own separate VLAN,
such communication is not possible without some remediation.

To solve some of this problem,
one can turn on IGMP Snooping and Multicast DNS (mDNS) between the necessary VLANs on a Ubiquiti Dream Machine (UDM).
Unfortunately, the UDM does not also support a necessary UPnP SSDP relay capability.

Fortunately, deployment of a
[multicast-relay](https://github.com/alsmith/multicast-relay)
solves this problem.
It seems that this used to be deployable on a UDM in a
[Docker container](https://github.com/scyto/multicast-relay).
That does not seem to be the case now.
A similar deployment might be done on a capable NAS but that would invite some exposure/risk from these VLANs.
Instead, it seems best to deploy a dedicated device for this purpose.

# Raspberry Pi Installation

I did this on my Fedora Linux machine but the same results can be achieved otherwise.

Download a [Raspberry Pi OS image](https://www.raspberrypi.com/software/operating-systems/).
Raspberry Pi OS Lite will suffice.

Use

	gnome-disks

to identify where the media is.
For example,

	media=/dev/sda

Write the downloaded image to the media

	xzcat 2023-12-05-raspios-bookworm-arm64.img.xz | sudo dd obs=1M oflag=direct of=$media

Resize the **rootfs** to consume all free space (using **gnome-disks**).
Mount **bootfs** and **rootfs**.

Enable **ssh** on boot.

	sudo touch /run/media/$USER/bootfs/ssh

Enable **pi** account login by replacing its impossible password hash ('!') with a known one (e.g. yours from /etc/shadow).

	sudo vim /run/media/$USER/rootfs/etc/shadow

Unmount **bootfs** and **rootfs** using **gnome-disks**.

# Add to Network

Configure the Raspberry Pi's switch port so that its default VLAN is that of the Sonos devices.
In addition add the VLAN tags of the network(s) of the Sonos controllers.

Attach the Raspberry Pi to the network and start it.

Optionally, give it a reserved DHCP address and DNS name (e.g. sonos-relay.home.arpa).

# Configuration

Login with ssh

	ssh pi@sonos-relay.home.auto

Git sonos-relay

	git clone https://github.com/rtyle/sonos-relay

	(cd sonos-relay; git submodule update --init --recursive)

Configure the VLAN interfaces

	nmcli con

Choose the interface. For example

	if=eth0

For each tagged VLAN

	nmcli con add type vlan con-name $name dev $if id $id ipv4 $ip gw4 $gw
Where
