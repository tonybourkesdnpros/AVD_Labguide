# Adding spine4

In this lab, you will add spine4 to the fabric simply by editing the inventory file and the FABRIC.yml file. 

<img src=lab6-images/1.png width="50%" height="50%" border=1>


## Edit inventory.yml

For AVD to work, it must know about the devices through the inventory file. Edit the inventory.yml file and add the following spine4 entry after spine3: 


<img src=lab6-images/2.png width="50%" height="50%" border=1>


<pre>
            spine4: 
              ansible_host: 192.168.0.14
</pre>

## Edit FABRIC.yml

The FABRIC.yml edits will be a bit more extensive. Open the FABRIC.yml file. 


<img src=lab6-images/3.png width="50%" height="50%" border=1>

Under the spines section, add the entry for spine4:

<img src=lab6-images/4.png width="50%" height="50%" border=1>

The text will be as follows:

<pre>
    - name: spine4
      id: 14
      mgmt_ip: 192.168.0.14/24
</pre>

Under "uplink_interfaces" add "Ethernet6" to the list of interfaces, and then add spine4 to the list of switches: 

<img src=lab6-images/5.png width="50%" height="50%" border=1>

And then under the node groups, add the spine uplink interfaces (the interfaces on the switches that connect to spine 4). 

<img src=lab6-images/6.png width="50%" border=1>

The FABRIC.yml configuration should look like this: 

<pre>
---

fabric_name: FABRIC

# Various fabric settings

# Enable vlan aware bundles
evpn_vlan_aware_bundles: false


# Spine Switches
spine:
  defaults:
#    platform: vEOS-LAB # 
    bgp_as: 65001
    loopback_ipv4_pool: 192.168.101.0/24
    bgp_defaults:
      - 'no bgp default ipv4-unicast'
      - 'distance bgp 20 200 200'
    mlag: false
  nodes:
    - name: spine1
      id: 11
      mgmt_ip: 192.168.0.11/24
    - name: spine2
      id: 12
      mgmt_ip: 192.168.0.12/24
    - name: spine3
      id: 13
      mgmt_ip: 192.168.0.13/24
    - name: spine4
      id: 14
      mgmt_ip: 192.168.0.14/24

# Leaf switches. Most leafs will be l3leaf, not l2leaf.
l3leaf:
  defaults:
    bgp_as: 65100-65199 # Gives a range which will be auto-assigned
#    platform: vEOS-LAB
    loopback_ipv4_pool: 192.168.101.0/24 # This is loopback0 (underlay)
    vtep_loopback_ipv4_pool: 192.168.102.0/24 # This is loopback1 (VTEP)
    uplink_interfaces: [Ethernet3, Ethernet4, Ethernet5, Ethernet6] # Leaf uplinks
    uplink_switches: [spine1, spine2, spine3, spine4] # Where the leaf uplinks go
    uplink_ipv4_pool: 192.168.103.0/24 # For the p2p interfaces to chopped up into /31s
    mlag_interfaces: [Ethernet1, Ethernet2] # MLAG peer link
    mlag_peer_ipv4_pool: 10.255.252.0/24 # MLAG peer IPs
    mlag_peer_l3_ipv4_pool: 10.255.251.0/24 # iBGP peering between MLAG peers
    virtual_router_mac_address: 00:1c:73:00:00:99 # The vMAC for the anycast gateways
    bgp_defaults:
      - 'no bgp default ipv4-unicast'
      - 'distance bgp 20 200 200'
      - 'graceful-restart restart-time 300'
      - 'graceful-restart'
    spanning_tree_mode: mstp # Spanning Tree is still enabled even in EVPN setups
    spanning_tree_priority: 16384 
    mlag: true # By default, use MLAG
  node_groups: 
    - group: mlag1
    # bgp_as is configured automatically from the range, but can be overriden
      nodes:
        - name: leaf1
          id: 1
          mgmt_ip: 192.168.0.21/24
          uplink_switch_interfaces: [Ethernet3, Ethernet3, Ethernet3, Ethernet3]
        - name: leaf2
          id: 2
          mgmt_ip: 192.168.0.22/24
          uplink_switch_interfaces: [Ethernet4, Ethernet4, Ethernet4, Ethernet4]
    - group: mlag2
      bgp_as: 65299 # Overriden automatic allocation for host4/R1 peering
      nodes:
        - name: leaf3
          id: 3
          mgmt_ip: 192.168.0.23/24
          uplink_switch_interfaces: [Ethernet5, Ethernet5, Ethernet5, Ethernet5]
        - name: leaf4
          id: 4
          mgmt_ip: 192.168.0.24/24
          uplink_switch_interfaces: [Ethernet6, Ethernet6, Ethernet6, Ethernet6]



# There's an issue with vEOS with larger MTUs
p2p_uplinks_mtu: 1500

# BFD Settings
bfd_multihop:
  interval: 1200
  min_rx: 1200
  multiplier: 3


# # If you want to put a password on peers
# bgp_peer_groups:
#   # all passwords set to "arista"
#   evpn_overlay_peers:
#     password: Q4fqtbqcZ7oQuKfuWtNGRQ==
#   ipv4_underlay_peers:
#     password: 7x4B4rnJhZB438m9+BrBfQ==
#   mlag_ipv4_underlay_peer:
#     password: 4b21pAdCvWeAqpcKDFMdWw==

# Needed for vEOS/cEOS

bgp_update_wait_install: false
bgp_update_wait_for_convergence: false

# Needed for Arista ATD Lab Environment
dns_domain: atd.lab
mgmt_interface: Management0
mgmt_interface_vrf: MGMT
mgmt_gateway: 192.168.0.1
</pre>

## Build and Deploy

Build and deploy the new configuration with <tt>build_fabric.yml</tt> and <tt>deploy_fabric.yml</tt>. Be sure to run any necessary tasks through a change control. 

Build the config: 
<pre>
➜  AVD_Labs git:(main) ✗ <b><span style="color:red;">ansible-playbook playbooks/build_fabric.yml</b></span>
</pre>

You should see in the intended/config directory the spine4.cfg file: 

<img src=lab6-images/7.png border=1>


Deploy the new config: 
<pre>
➜  AVD_L3LS git:(main) ✗ <b><span style="color:red;">ansible-playbook playbooks/deploy_fabric.yml</b></span>
</pre>