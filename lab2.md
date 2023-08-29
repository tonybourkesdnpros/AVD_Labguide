# Setting up Arista AVD

Arista AVD is an open source tool for the creation of various types of fabric configurations as well as deployment options, automatic documentation, and post-deployment validation. With just a few data models, AVD can generate complex EVPN/VXLAN configurations for dozens (or even hundreds) of switches. It's also capable of generating traditional L2LS (collapsed core without EVPN) as well as MPLS topologies. 

AVD primarily utilizes Ansible to accomplish this by providing an advanced set of playbooks, data models, templates, roles, and modules. 

In this lab, you will create an L3LS+EVPN/VXLAN configuration with just a few files. 

## Prepare Data Models

Select the AVD_Labs directory. You'll see a directory called AVD_L3LS, Ansible_CVP, Ansible_EOS, and AVD_L3LS_MultiDC. Plus a few other administrative files (LICENSE, README.md, etc.)

<img src=lab2-images/1.png border=1>

From there, select the AVD_L3LS directory. 

You'll see a few directories and files:

* group_vars
* playbooks
* ansible.cfg
* inventory.yml

<img src=lab2-images/2.png border=1>


Group_vars will have the data models used to build the fabric configuration files. 

Playbooks will have the various playbooks being used (build, deploy, test).

The ansible.cfg file has the minimum parameters for Ansible to work with AVD. 

The inventory will have the individual devices, grouped according to their roles and the network topology. 

Feel free to explore the files. 

## Inventory File

For Ansible to run, there must be an inventory file. This file will have the login information for both CloudVision Portal and the individual leafs and spines. The former is used to upload and apply the generated configurations, and the later is used to know which devices AVD should generate configurations for. 

Open the inventory file in the editor by clicking on inventory.yml. 

<img src=lab2-images/3.png border=1>


There is the "all" group, which is all of the groups. "Children" signifies that there are more groups (as opposed to hosts). 

There is a group called <b>CVP_cluster</b> which is where you would put all of the CVP hosts. In the lab environment, there is only one CVP host, named cvp1. 

Change the <tt>ansible_password</tt>: field (highlighted in red) to your environment's password. It will be "arista" followed by four alphanumeric characters. 

<pre>
---
all:
  children:
    CVP_cluster:
      hosts: 
        cvp1: 
          ansible_httpapi_host: 192.168.0.5
          ansible_host: 192.168.0.5
          ansible_user: arista
          ansible_password: <span style="color:red;"><b>aristaXXXX</b></span>
          ansible_connection: httpapi
          ansible_httpapi_use_ssl: True
          ansible_httpapi_validate_certs: False
          ansible_network_os: eos
          ansible_httpapi_port: 443
</pre>

* <i>Note: While the passwords are located in the inventory file for convenience sake, there are methods to encrypt the password using mechanisms like Ansible Vault to ensure that no password is show in plaintext</i>

You can explore the rest of the inventory file, which will have a FABRIC group, as well as sub groups. You do not need to modify anything at this time. 

* FABRIC
  * SPINES
  * LEAFS
* EVPN_SERVICES
* ENDPOINT_CONNECT


## Build EVPN/VXLAN 

With AVD, there will be three playbooks used. 

* build_fabric.yml: This builds and documents the configuration
* deploy_fabric.yml: This deploys the configuration (through CVP)
* test_fabric.yml: This tests the deployed environment

The build_fabric.yml playbook is the first one we will use. It will both build configlets for the fabric, as well as create documentation for that build. 

The build process will take the three data models. You'll find them in the group_vars directory: 

* FABRIC.yml: This file describes the overall fabric (for a single DC environment, this includes all the leafs and spines)
* EVPN_SERVICES.yml: This file describes the VXLAN segments and anycast gateways to be created
* ENDPOINT_CONNECT.yml: This file describes how the hosts will be connected to the network through the leafs

AVD will use Ansible to take these data models and convert them into individual configlets for the leafs and spines. 

The deploy process will take those configlets, upload them to CloudVision, and attach them to the various devices. This will generate tasks that the operator can run through a change control. 

After the change control process has been completed, then the test playbook will run a series of tests on all of the devices, such as checking each device's routing table to make sure the loopbacks are present. 

### Build AVD Configuration

Change directory to the AVD_L3LS directory. 

<pre>
➜  project <span style="color:red;"><b>cd labfiles</b></span>
➜  labfiles <span style="color:red;"><b>cd AVD_Lab</b></span>
➜  AVD_Lab git:(main) <span style="color:red;"><b>cd AVD_L3LS</b></span>
</pre>

Verify you're in the directory of <tt>/home/coder/project/labfiles/AVD_Lab/AVD_L3LS</tt> by running the <tt>pwd</tt> command, which will show you your current directory. 

<pre>
➜  AVD_L3LS git:(main) <span style="color:red;"><b>pwd</b></span>
/home/coder/project/labfiles/AVD_Lab/AVD_L3LS
</pre>

Run the build_fabric.yml playbook. The output below has been truncated. 

<pre>
  AVD_L3LS git:(main) ✗ <span style="color:red;"><b>ansible-playbook playbooks/build-fabric.yml</b></span>
...
PLAY RECAP *******************************************************************************************************
leaf1                      : ok=3    changed=3    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
leaf2                      : ok=3    changed=3    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
leaf3                      : ok=3    changed=3    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
leaf4                      : ok=3    changed=3    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
spine1                     : ok=11   changed=8    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   
spine2                     : ok=3    changed=3    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
spine3                     : ok=3    changed=3    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
</pre>

This will create new directories called "intended" and "documentation" and will populate them with files. The "intended" directory contains both the intermediary EOS config (in YAML) in a directory called "structured_configs" as well as "configs" which contain the configlets that will be uploaded. 

The structured_config files are a YAML representation of the desired configuration state, which is used (automatically in this case) to generate the EOS syntax files under "intended". 

<img src=lab2-images/4.png border=1>

You can click on them to explore the configurations generated. leaf1.cfg, for example, is over 200 lines of newly generated configuration based off the data models.

<img src=lab2-images/5.png border=1>


## View the Configuration Documentation

Click on the documentation directory and open up the fabric subdirectory. Right click on the FABRIC-documentation.md file and click "Open preview". 

<img src=lab2-images/6.png border=1>

This will show you the report that AVD automatically generates when creating configurations. 

This is a markdown file, a kind of simple HTML page. Click on the "Fabric Point-To-Point Links" link. 

<img src=lab2-images/7.png border=1>

In the FABRIC.yml file, there was this line under spine:

<tt>    loopback_ipv4_pool: 192.168.101.0/24</tt>

and the same under leaf:

<tt>    loopback_ipv4_pool: 192.168.101.0/24</tt>

AVD automatically divided up the /24 into 128 /31s and auto-assigned them to the point-to-point links between the leafs and spines. The "Fabric Point-To-Point Links" sections shows how many of those /31s were consumed when building the configuration. 

The report states how much of the /31s have been consumed by existing point-to-point links. 


## Deploy Configurations

Now deploy the configurations to CloudVision with the command <span style="color:red;"><tt>ansible-playbook playbooks/deploy_fabric.yml</tt></span>

<pre>
  AVD git:(main) ✗ <span style="color:red;"><b>ansible-playbook playbooks/deploy_fabric.yml</b></span>
...

TASK [arista.avd.eos_config_deploy_cvp : Configure devices on cvp1] ****************************************************************************************************************************************
changed: [cvp1]

TASK [arista.avd.eos_config_deploy_cvp : Execute pending tasks on cvp1] ************************************************************************************************************************************
skipping: [cvp1]

PLAY RECAP *************************************************************************************************************************************************************************************************
cvp1                       : ok=10   changed=1    unreachable=0    failed=0    skipped=3    rescued=0    ignored=0   

➜  AVD_L3LS git:(main) 
</pre>

This will create 7 tasks for the leafs and spines. 

On the top bar, select "Provisioning", select "Tasks" on the left menu, and click on the "ID" check box to select all the tasks. 

<img src=lab2-images/9.png border=1>

Click on the "Tasks" section on the sidebar 

<img src=lab2-images/5.png border=1>

The tasks represent a potential change in configuration reflecting the new configlets applied to each device. The change isn't implemented until a change control has been run. 

### Run Change Control

Under Provisioning and Tasks, select all the tasks by clicking the button at the top, and then click "+ Create Change Control". 

<img src=lab2-images/6.png border=1>

Change the arrangement to "Parallel", and click "Create Change Control with 6 Tasks"

<img src=lab2-images/7.png border=1>

This will bring you to a new change control. Click on "Review and Approve" in the upper right. 

<img src=lab2-images/8.png border=1>

You can scroll through the various proposed changes. The change control can only be executed when it has been approved. Click on "Approve" in the lower right hand corner. 

<img src=lab2-images/9.png border=1>

This will bring you back to the change control page. Click on the "Execute Change Control" button in the upper right hand corner. 

<img src=lab2-images/10.png border=1>

Click the "Execute" button in the confirmation window. 

<img src=lab2-images/11.png border=1>

The change control process will start. It usually will complete in less than 30 seconds, though it may take longer depending on how busy the server is. 

When it's completed, you'll see green checks for each device, and the status will show a green "Completed". 

<img src=lab2-images/12.png border=1>

### Verify Change on Leaf1-DC1

From the command line, SSH into leaf1 with the command <tt>ssh arista@leaf1</tt>. You should be allowed in without a password prompt (the switches have an SSH key installed).

<pre>
 ➜  AVD_L3LS git:(main) ssh leaf1
Last login: Sat Aug 26 13:19:53 2023 from 192.168.0.1
leaf1#
</pre>

Run the command <tt>show ip bgp summary</tt> to see if the underlay has been configured. You should see four "Estab" sessions: Three to the spines, and one to leaf2. 


<pre> 
leaf1#<span style="color:red;">show ip bgp summary</span>
BGP summary information for VRF default
Router identifier 192.168.101.1, local AS number 65100
Neighbor Status Codes: m - Under maintenance
  Description              Neighbor      V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  leaf2                    10.255.251.1  4 65100              8         3    0    0 00:00:01 Estab   8      8
  spine1_Ethernet3         192.168.103.0 4 65001              8         4    0    0 00:00:03 Estab   3      3
  spine2_Ethernet3         192.168.103.2 4 65001             10         3    0    0 00:00:02 Estab   0      0
  spine3_Ethernet3         192.168.103.4 4 65001              8        10    0    0 00:00:03 Estab   0      0
</pre>

Check the overlay peering with the command <tt>show bgp evpn summary</tt>. You should see three "Estab" sessions with the spines. 

<pre>
leaf1#<b><span style="color:red;">show bgp evpn summary</span></b>
BGP summary information for VRF default
Router identifier 192.168.101.1, local AS number 65100
Neighbor Status Codes: m - Under maintenance
  Description              Neighbor       V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  spine1                   192.168.101.11 4 65001             30        28    0    0 00:14:31 Estab   4      4
  spine2                   192.168.101.12 4 65001             30        30    0    0 00:14:30 Estab   4      4
  spine3                   192.168.101.13 4 65001             30        27    0    0 00:14:30 Estab   4      4

</pre>

## Configure host1

Log into host1 (You can open a new terminal session with the "+" button on the upper right).

<img src=lab3-images/13.png border=1>

Configure host1's Ethernet1 and Ethernet2 into a Layer 3 port channel with the IP address of 10.1.10.11/24 and default gateway of 10.1.10.1. 
<pre>
➜  project <b><span style="color:red;">ssh host1</span></b>
host1#<b><span style="color:red;">conf</span></b>
host1(config)#<b><span style="color:red;">int e1-2</span></b>
host1(config-if-Et1-2)#<b><span style="color:red;">channel-group 1 mode active </span></b>
host1(config-if-Et1-2)#<b><span style="color:red;">int po1</span></b>
host1(config-if-Po1)#<b><span style="color:red;">no switchport </span></b>
host1(config-if-Po1)#<b><span style="color:red;">ip address 10.1.10.11/24</span></b>
host1(config-if-Po1)#<b><span style="color:red;">ip route 0.0.0.0/0 10.1.10.1</span></b>
</pre>

You should be able to ping the default gateway now. 

<pre>
host1(config)#<b><span style="color:red;">ping 10.1.10.1</span></b>
PING 10.1.10.1 (10.1.10.1) 72(100) bytes of data.
80 bytes from 10.1.10.1: icmp_seq=1 ttl=64 time=5.71 ms
80 bytes from 10.1.10.1: icmp_seq=2 ttl=64 time=5.97 ms
80 bytes from 10.1.10.1: icmp_seq=3 ttl=64 time=4.51 ms
80 bytes from 10.1.10.1: icmp_seq=4 ttl=64 time=3.10 ms
80 bytes from 10.1.10.1: icmp_seq=5 ttl=64 time=3.28 ms

--- 10.1.10.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 23ms
rtt min/avg/max/mdev = 3.107/4.519/5.974/1.189 ms, ipg/ewma 5.766/5.031 ms
</pre>
