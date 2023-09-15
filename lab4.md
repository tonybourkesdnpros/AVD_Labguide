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

The playbook will write its report as a MD (markdown) file in the reports directly. 

When the playbook is complete, find the "reports" directory and right-click the FABRIC-state.md file and click "Open Preview". 

<img src=lab4-images/1.png border=1>

This will bring up the report. The first page of the report should show zero errors (if there are errors, you can investigate). Over 200 tests were performed. 

<img src=lab4-images/2.png border=1>

Scroll down to the section "Summary Totals Per Category" and see the various categories that are tested. 

<img src=lab4-images/3.png border=1>




