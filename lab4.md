# Validate Configuraiton

One of the capabilities of AVD is to validate/test configurations that were generated and deployed by AVD. 

## Validate

Run the the <tt>test_fabric.yml</tt> playbook. This playbook may take a minute or two to complete. 

<pre>
➜  AVD_L3LS git:(main) ✗ <b><span style="color:red;">ansible-playbook playbooks/test_fabric.yml</span></b>
PLAY RECAP *************************************************************************************************************************************************************************************************
leaf1                      : ok=38   changed=0    unreachable=0    failed=0    skipped=4    rescued=0    ignored=0   
leaf2                      : ok=38   changed=0    unreachable=0    failed=0    skipped=4    rescued=0    ignored=0   
leaf3                      : ok=38   changed=0    unreachable=0    failed=0    skipped=4    rescued=0    ignored=0   
leaf4                      : ok=38   changed=0    unreachable=0    failed=0    skipped=4    rescued=0    ignored=0   
spine1                     : ok=34   changed=6    unreachable=0    failed=0    skipped=14   rescued=0    ignored=0   
spine2                     : ok=26   changed=0    unreachable=0    failed=0    skipped=14   rescued=0    ignored=0   
spine3                     : ok=26   changed=0    unreachable=0    failed=0    skipped=14   rescued=0    ignored=0   
</pre>



