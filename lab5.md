# Connect to Outside Network

In this lab, host4 will be configured as if it were an external router (we'll call it "R1"), and will exchange routes to and from the fabric. 

<img src=lab5-images/R1_endstate.png width="50%" height="50%" border=1>


## Configure the Fabric to Peer with R1

Open the file under group_vars called EVPN_SERVICES.yml. Under the Tenant "ACME" and VRF "VRF_A", add the following lines after the last line in the file: 

<pre>
        l3_interfaces:
          - interfaces: [Ethernet9]
            ip_addresses: [10.1.5.1/24]
            nodes: [leaf4]
            enabled: True 
        bgp_peers:
          - ip_address: 10.1.5.254
            remote_as: 1
            nodes: [leaf4]
</pre>

The file EVPN_SERVICES.yml should look like this when complete: 

<pre>
---
tenants:
  - name: ACME
    mac_vrf_vni_base: 10000
    vrfs:
      - name: VRF_A
        vrf_vni: 10
        svis:
          - id: 10
            # SVI Description
            name: DMZ
            enabled: true
            ip_address_virtual: 10.1.10.1/24
          - id: 20
            name: Internal
            enabled: true
            ip_address_virtual: 10.1.20.1/24
        l3_interfaces:
          - interfaces: [Ethernet9]
            ip_addresses: [10.1.5.1/24]
            nodes: [leaf4]
            enabled: True 
        bgp_peers:
          - ip_address: 10.1.5.254
            remote_as: 1
            nodes: [leaf4]
</pre>

Because the data model changed, run the build_config.yml playbook, and then the deploy_config.yml playbook. 

<pre>
AVD_L3LS git:(main) ✗ <b><span style="color:red;">ansible-playbook playbooks/build_fabric.yml</span></b>
...
PLAY RECAP *****************************************************************************************************************************************
leaf1                      : ok=3    changed=2    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
leaf2                      : ok=3    changed=2    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
leaf3                      : ok=3    changed=2    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
leaf4                      : ok=3    changed=2    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
spine1                     : ok=11   changed=7    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   
spine2                     : ok=3    changed=2    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
spine3                     : ok=3    changed=2    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
spine4                     : ok=3    changed=2    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
</pre>

Then run the deploy_fabric.yml playbook. 

<pre>
AVD_L3LS git:(L3v2) ✗  <b><span style="color:red;">ansible-playbook playbooks/deploy_fabric.yml</b></span>

PLAY [Deploy fabric to CVP] *******************************************************************************************************
...
PLAY RECAP ************************************************************************************************************************
cvp1                       : ok=11   changed=2    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   
</pre>

Run any change controls needed to push the change to the devices. 

After the change controls are complete, log into leaf4 to verify that the configuration change was completed correctly. Run the command <tt>show ip bgp summary vrf VRF_A</tt>

<pre>
leaf4#<b><span style="color:red;">show ip bgp summary vrf VRF_A</b></span>
BGP summary information for VRF VRF_A
Router identifier 192.168.101.4, local AS number 65103
Neighbor Status Codes: m - Under maintenance
  Neighbor   V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.1.5.254 4 1                  0         0    0    0 00:02:18 <b>Active</b>
</pre>

You should see a single session in the "Active" state. It will stay in this state until "R1" is configured to peer. 

## Configure "R1"

Log into host4 and configure it as a router. Paste the following configuration (while in config mode):

<pre>
hostname R1

ip routing

interface Ethernet2
   no switchport
   ip address 10.1.5.254/24

interface Vlan1000
   no autostate
   ip address 172.16.0.1/12


router bgp 1
   router-id 9.9.9.9
   neighbor 10.1.5.1 remote-as 65299
   !
   address-family ipv4
      neighbor 10.1.5.1 activate
      network 172.16.0.0/12
</pre>



Once this is complete, check to see if R1 is peering with leaf4. 

<pre>
R1(config)#<b><span style="color:red;">show ip bgp summary</span></b>
BGP summary information for VRF default
Router identifier 9.9.9.9, local AS number 1
Neighbor Status Codes: m - Under maintenance
  Neighbor V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.1.5.1 4 65299             26        32    0    0 00:09:21 <b>Estab</b>   5      5
</pre>

Check to see if R1 has received the routes from the EVPN fabric. 

<pre>
R1(config)#<b><span style="color:red;">show ip route</b></span>

VRF: default
Codes: C - connected, S - static, K - kernel, 
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1,
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1,
       N2 - OSPF NSSA external type2, B - Other BGP Routes,
       B I - iBGP, B E - eBGP, R - RIP, I L1 - IS-IS level 1,
       I L2 - IS-IS level 2, O3 - OSPFv3, A B - BGP Aggregate,
       A O - OSPF Summary, NG - Nexthop Group Static Route,
       V - VXLAN Control Service, M - Martian,
       DH - DHCP client installed default route,
       DP - Dynamic Policy Route, L - VRF Leaked,
       G  - gRIBI, RC - Route Cache Route,
       CL - CBF Leaked Route

Gateway of last resort is not set

 C        10.1.5.0/24 is directly connected, Ethernet2
 B E      <b>10.1.10.0/24</b> [200/0] via 10.1.5.1, Ethernet2
 B E      <b>10.1.20.0/24</b> [200/0] via 10.1.5.1, Ethernet2
 B E      10.255.251.0/31 [200/0] via 10.1.5.1, Ethernet2
 C        172.16.0.0/12 is directly connected, Vlan1000
</pre>

You should see both 10.1.10.0/24 and 10.1.20.0/24 from the ACME tenant on the fabric. 

Log into leaf1 and check the routes for the VRF. 

<pre>
leaf1#<b>show ip route vrf VRF_A </b>

VRF: VRF_A
Codes: C - connected, S - static, K - kernel, 
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1,
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1,
       N2 - OSPF NSSA external type2, B - Other BGP Routes,
       B I - iBGP, B E - eBGP, R - RIP, I L1 - IS-IS level 1,
       I L2 - IS-IS level 2, O3 - OSPFv3, A B - BGP Aggregate,
       A O - OSPF Summary, NG - Nexthop Group Static Route,
       V - VXLAN Control Service, M - Martian,
       DH - DHCP client installed default route,
       DP - Dynamic Policy Route, L - VRF Leaked,
       G  - gRIBI, RC - Route Cache Route,
       CL - CBF Leaked Route

Gateway of last resort is not set

 B E      10.1.5.0/24 [20/0] via VTEP 192.168.102.4 VNI 10 router-mac 00:1c:73:c4:c6:01 local-interface Vxlan1
 C        10.1.10.0/24 is directly connected, Vlan10
 C        10.1.20.0/24 is directly connected, Vlan20
 C        10.255.251.0/31 is directly connected, Vlan3009
 B E      <b>172.16.0.0/12</b> [20/0] via VTEP 192.168.102.4 VNI 10 router-mac 00:1c:73:c4:c6:01 local-interface Vxlan1
</pre>

Log into host1 and try to ping 172.16.0.1 (the IP address of VLAN 1000 on R1): 

<pre>
➜  project <b><span style="color:red;">ssh host1</span></b>
Last login: Sun Sep 10 12:52:53 2023 from 192.168.0.1
host1#ping 172.16.0.1
PING 172.16.0.1 (172.16.0.1) 72(100) bytes of data.
80 bytes from 172.16.0.1: icmp_seq=1 ttl=63 time=13.8 ms
80 bytes from 172.16.0.1: icmp_seq=2 ttl=62 time=5.20 ms
80 bytes from 172.16.0.1: icmp_seq=3 ttl=62 time=5.44 ms
80 bytes from 172.16.0.1: icmp_seq=4 ttl=62 time=5.44 ms
80 bytes from 172.16.0.1: icmp_seq=5 ttl=62 time=5.18 ms

--- 172.16.0.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 46ms
rtt min/avg/max/mdev = 5.183/7.025/13.845/3.412 ms, pipe 2, ipg/ewma 11.533/10.316 ms
</pre>