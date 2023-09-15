# Reset Lab Environment

This lab will reset the devices in the lab to their default configurations/configlets/containers. You can run this lab at any time, and it will apply the ATD-INFRA and device-base configets and remove any others that have been applied (you may need to manually remove any configlets that were applied to containers).

## Run Reset Playbook 

Make sure you are in the AVD_L3LS directory under AVD_Labs

<pre>
labfiles <span style="color:red;"><b>cd AVD_Labs</span></b>
➜  AVD_Labs git:(main) ✗ <span style="color:red;"><b>cd AVD_L3LS</b></span>
</pre>

Run the reset playbook

<pre>
➜  AVD_L3LS git:(main) ✗ <span style="color:red;">ansible-playbook reset/playbook/CVP_reset.yml</b></span>
...
TASK [Apply configlets for default config] ************************************************************************************
changed: [cvp1]

PLAY RECAP ************************************************************************************
cvp1                       : ok=2    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
</pre>

When this is completed, run any tasks through a normal change control. 

<img src=lab1.5-images/reset2.png border=1>

