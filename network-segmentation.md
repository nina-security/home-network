# Network Segmentation
[This is WIP]

As I wanted to restrict access between my network devices, I came up with at network segmentation concept containing different subnets/VLANs for different types of devices.
For example, my work devices have a dedicated network with no traffic allowed in or out.
Also, between different subnets specific traffic is allowed to pass as configured in the firewall rules. 

# Network Segmentation Test

Network segmentation tests check if the network separation is as intended by the configured firewall rules.

First, to check which ports are exposed on my devices, I ran a network scan from a subnet having access to all other subnets.  
This helped me evaluating which firewall rules I need to have required network communication between devices/subnets working as desired. 

Moreover, I now had a detailed list of reachable network services which I could use as a basis for targeted segmentation tests. For this aim, I ran a network scan from each subnet to all other available IPs and ports identified in the initial scan.

Note that below in the scans only TCP ports are considered. UDP ports should also be checked.

## Configure Scan Device

For the segmentation test, I configured a Raspberry Pi to be connected to all possible VLANs using a trunk port.

Suppose that the Raspberry Pi is connected on the network interface `eth0` to its default subnet `192.168.1.0/24`.
Then these commands configure a VLAN interface for another existing VLAN with ID `10`:

```bash
sudo ip link add link eth0 name eth0.10 type vlan id 10
sudo ip link set eth0.10 up
```

To delete an interface after scanning, run:
```bash
sudo ip link delete eth0.10
```

## Scans

### Identify Reachable Systems & Ports

One of my networks, to which the Raspberry Pi belongs natively, has network access to all other subnets.
Run the following command to identify all available TCP services. Make sure devices are up, as they cannot be scanned otherwise. 

The file `ips.txt` contains a list with all network ranges used on my network.

```bash
sudo nmap -Pn -A -T4 -iL ips.txt -p- -oA src_raspberry-dst_all -vvv
```

### Scan From a Specific VLAN

Below is a sample nmap command to scan from a specific source interface (`eth0.10`). 
Here, the file `ips.txt` is more targeted and contains a list only with IP addresses in use.
Moreover, the port argument `-p` contains only ports which were reachable in the intital scan. 

```bash
sudo nmap -Pn -A -T4 -iL ips.txt -p x,y,z -oA src_vlan_10-dst_all -e eth0.10 -vvv
```