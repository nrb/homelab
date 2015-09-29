# homelab
A place to keep all my notes and code for my home lab.

# The Hardware

The lab will use the server `odin` to host the lab's infrastructure, like DHCP and imaging services.

There are currently 5 nodes.

The main switch is a Cisco Catalyst 2950.

## Odin

The `odin` server has the following hardware:

    * Intel Core i3-4130T CPU @ 2.90GHz
    * 8 GB RAM
    * 256GB M2 SATA3 SSD
    * 2 integrated NICs (`p2p1` and `p3p1` in Linux)


## Lab Nodes

5 nodes, with the following in common:

    * HP G7 NG54L microservers
    * 500 GB hard drives
    * 3 total NICs (2 PCIe NICs per machine)

3 nodes with 16GB RAM
2 nodes with 8GB RAM

There are 5 nodes that can be used for the actual lab itself. They are all HP G7 N54Ls, currently all with 500GB RAM.
3 of them have 16GB RAM, the other 2 have 8GB. Each machine also has 2 PCIe NICs added, bringing the total to 3 per machine.

# Network Setup

`odin` is joined to both the home network (192.168.2.0/24) and the lab network (172.16.0.0/24).

On the lab network, `odin` uses a Linux bridge to provide any containers it might host access to the hardware network.

The bridge is named `br-lab`.

## Enable IP forwarding in the kernel

In order to send traffic from the lab network, IPv4 forwarding needs to be enabled in the kernel.

This is accomplished like so:

`echo net.ipv4.ip_forward=1 >> /etc/sysctl.conf`
`sysctl -p`

This will let the kernel pass traffic from different interfaces and subnets between each other.

## Enable packet forwarding with iptables

Next, the kernel needs to know exactly how to route packets between the two networks.

For now, I've done so wih the following iptables rules.

This one enables NAT on the 'external" (to the lab, anyway) interface:

    `iptables -t nat -A POSTROUTING -o p2p1 -j MASQUERADE`

Here, we tell the kernel that we should allow traffic from the lab's subnet on the `br-lab` interface to be forwarded:

    `iptables -A FORWARD -s 172.16.0.0/24 -i br-lab -j ACCEPT`

Note that these are NOT saved with these commands; they're merely added to the running system.

This is not yet written out as I would like to ensure that the things `lxc` does to iptables to make its own `lxcbr0` bridge
work is applied to `br-lab`.
