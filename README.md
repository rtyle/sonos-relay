# Sonos Inter-VLAN Relay

A Sonos Inter-VLAN Relay (sonos-relay) may be necessary when Sonos devices are on a separate VLAN from their controllers.
This repository documents the deployment of a [multicast-relay](https://github.com/alsmith/multicast-relay) on a Raspberry Pi dedicated for this purpose.

# Best Networking Practices for Sonos Devices

Guidance given
[here](https://help.ui.com/hc/en-us/articles/18930473041047-Best-Practices-for-Sonos-Devices)
helps to avoid networking problems with Sonos devices.

It is best to create a VLAN dedicated exclusively to Sonos devices.
Wire all Sonos devices into this VLAN.
Because they are wired, they do not mesh wirelessly (using SonosNet) which,
if not handled carefully with the right
[STP](https://en.wikipedia.org/wiki/Spanning_Tree_Protocol)
configuration, can create disasterous network loops.

Do not use the extra port in the embedded switches of some Sonos devices.

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

This describes the installation from a Fedora Linux machine.
The same results can be achieved otherwise.

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
Add the tagged VLANs of the Sonos controllers.

Attach the Raspberry Pi to the network and start it.

Optionally, give it a reserved DHCP address and DNS name (e.g. sonos-relay.home.arpa).

# Configuration

Because the routing configuration of the Raspberry Pi will change during the configuration described here,
it is best to (temporarily?) place the machine you will be accessing it from on the same VLAN
(by default or with a VLAN tagged interface).

Login

	ssh pi@sonos-relay.home.arpa

Set timezone.
For example,

	timedatectl list-timezones
 	sudo timedatectl set-timezone America/Los_Angeles

Git sonos-relay

	# git clone git@github.com:rtyle/sonos-relay
	git clone https://github.com/rtyle/sonos-relay
	(cd sonos-relay; git submodule update --init --recursive)

Configure the VLAN interfaces.
For each tagged VLAN,

	nmcli con add type vlan con-name $name dev eth0 id $id ip4 $ip gw4 $gw
 
Where $name is the name of the VLAN, $id is its VLAN ID, $ip is its IPv4 CIDR address and $gw is the gateway to this VLAN.

Install dependencies

	sudo apt install python3-netifaces

# Test Run

	sudo python /home/pi/sonos-relay/multicast-relay/multicast-relay.py --interfaces $(nmcli con | tail -n +2 | awk '{print $NF}' | grep eth0) --noMDNS --foreground --verbose

# Automatic Run

Enable the service. It will start automatically on reboot.

	sudo systemctl enable /home/pi/sonos-relay/sonos-relay.service
 
Start it now

 	sudo systemctl start sonos-relay

Status

  	systemctl status sonos-relay

Follow verbose logging (add --verbose to unit file)

   	journalctl -u sonos-relay -f
