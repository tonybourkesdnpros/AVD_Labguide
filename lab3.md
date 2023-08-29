# Add Second Network and Second Host

In this lab, you will edit the data models to add a new host (host2) and a new network (VLAN 20). 

Host2's Ethernet1 and Ethernet2 are connected to leaf1 and leaf2 on Ethernet9. 

<img src=lab3-images/host2.png border=1>


## Edit EVPN Services Data Model

The file <tt>EVPN_SERVICE.yml</tt> is responsible for the Tenant, VRF, VLANs, anycast gateway, etc., configurations. These are the network services that are provided to the endpoint. 

Open EVPN_SERVICE.yml, which can be found in the group_vars directory. 

<img src=lab3-images/1.png border=1>

In the file there is one Tenant with one VRF and one SVI/VLAN, VLAN 10. 

Add the following to the file, just after the last line

<pre>
          - id: 20
            name: Internal
            enabled: true
            ip_address_virtual: 10.1.20.1/24
</pre>

This will add a new VLAN/anycast gateway to the Tenant/VRF. 
It should look like this when complete: 

<pre>
---
tenants:
  - name: ACME
    mac_vrf_vni_base: 10000
    vrfs:
      - name: VRF_A
        vrf_vni: 10
        redistribute_mlag_ibgp_peering_vrfs: false
        svis:
          - id: 10
            # SVI Description
            name: DMZ
            enabled: true
            ip_address_virtual: 10.1.10.1/24
<b><span style="color:red;">          - id: 20
            name: Internal
            enabled: true
            ip_address_virtual: 10.1.20.1/24</b></span>
</pre>

Then open ENDPOINT_CONNECT.yml. 

<img src=lab3-images/2.png border=1>

Add the following to configure the interfaces that host2 is connected to. 

<pre>
  - name: host2
    adapters:
      - endpoint_ports: [ Ethernet1, Ethernet2 ]
        switch_ports: [ Ethernet9, Ethernet9 ]
        switches: [ leaf1, leaf2 ]
        vlans: 20
        mode: access
        spanning_tree_portfast: edge
        port_channel:
          description: PortChannel host2
          mode: active
</pre>

When completed ENDPOINT_CONNECT.yml should look like this in its entirety: 

<pre>
---
servers:
  - name: host1
    adapters:
      - endpoint_ports: [ Ethernet1, Ethernet2 ]
        switch_ports: [ Ethernet7, Ethernet7 ]
        switches: [ leaf1, leaf2 ]
        vlans: 10
        mode: access
        spanning_tree_portfast: edge
        port_channel:
          description: PortChannel host1
          mode: active
  - name: host2
    adapters:
      - endpoint_ports: [ Ethernet1, Ethernet2 ]
        switch_ports: [ Ethernet9, Ethernet9 ]
        switches: [ leaf1, leaf2 ]
        vlans: 20
        mode: access
        spanning_tree_portfast: edge
        port_channel:
          description: PortChannel host2
          mode: active
</pre>

## Build the Config

Now it's time to build the configuration again. Run the build_fabric playbook.

<pre>
➜  AVD_L3LS git:(main) ✗ ansible-playbook playbooks/build_fabric.yml 


PLAY RECAP ********************************************************************************************************
leaf1                      : ok=3    changed=3    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
leaf2                      : ok=3    changed=3    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
leaf3                      : ok=3    changed=3    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
leaf4                      : ok=3    changed=3    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
spine1                     : ok=11   changed=1    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   
spine2                     : ok=3    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
spine3                     : ok=3    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   

</pre>

## Deploy Fabric

Now that the configurations are built, it's time to deploy. Run the command <tt>ansible-playbook playbooks/deploy_fabric.yml</tt>

<pre>
...

TASK [arista.avd.eos_config_deploy_cvp : Configure devices on cvp1] ****************************************************************************************************************************************
ok: [cvp1]

TASK [arista.avd.eos_config_deploy_cvp : Execute pending tasks on cvp1] ************************************************************************************************************************************
skipping: [cvp1]

PLAY RECAP **************************************************************************************************************************************
cvp1                       : ok=10   changed=1    unreachable=0    failed=0    skipped=3    rescued=0    ignored=0     
</pre>

Check with CloudVision, there should be 


## Configure Host2

Log into host2. 

<pre>
➜  project ssh host2
host2#conf
host2(config)#int e1-2
host2(config-if-Et1-2)#channel-group 1 mode active 
host2(config-if-Et1-2)#int po1
host2(config-if-Po1)#no switchport 
host2(config-if-Po1)#ip address 10.1.20.12/24
host2(config-if-Po1)#ip route 0.0.0.0/0 10.1.20.1
</pre>

Save the config. 

<pre>
host2(config)#wr
Copy completed successfully.
</pre>

Test 
<pre>
host2(config)#ping 10.1.20.1
PING 10.1.20.1 (10.1.20.1) 72(100) bytes of data.
80 bytes from 10.1.20.1: icmp_seq=1 ttl=64 time=2.37 ms
80 bytes from 10.1.20.1: icmp_seq=2 ttl=64 time=1.14 ms
80 bytes from 10.1.20.1: icmp_seq=3 ttl=64 time=1.17 ms
80 bytes from 10.1.20.1: icmp_seq=4 ttl=64 time=1.20 ms
80 bytes from 10.1.20.1: icmp_seq=5 ttl=64 time=0.988 ms

--- 10.1.20.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 8ms
rtt min/avg/max/mdev = 0.988/1.376/2.370/0.502 ms, ipg/ewma 2.120/1.852 ms

</pre>