<!-- Output copied to clipboard! -->


Demo Debugging Ansible Playbooks

Saturday, 18 December 2021

12:32 pm

 



    * Debugging Ansible Playbooks

 



    * Ansible has a built-in debugger tool to troubleshoot data parsing/manipulation issues while running the playbook.
    * Below are the five different values that can be used to enable debugging in a playbook.

     


    [https://docs.ansible.com/ansible/latest/user_guide/playbooks_debugger.html#id13](https://docs.ansible.com/ansible/latest/user_guide/playbooks_debugger.html#id13)


     

    * never: Never enter the debugger 
    * always: Always enter the debugger 
    * on_failed : Enter debugger when task fails 
    * on_unreachable: Enter debugger when host can't be reached 
    * on_skipped : Enter debugger when the host is skipped by task 

     


    **Available debug commands:**


<table>
  <tr>
   <td>
<strong>Command</strong>
   </td>
   <td><strong>Shortcut</strong>
   </td>
   <td><strong>Action</strong>
   </td>
  </tr>
  <tr>
   <td><strong>print</strong>
   </td>
   <td>p
   </td>
   <td>Print information about the task
   </td>
  </tr>
  <tr>
   <td><strong>task.args[<em>key</em>] = <em>value</em></strong>
   </td>
   <td>no shortcut
   </td>
   <td>Update module arguments
   </td>
  </tr>
  <tr>
   <td><strong>task_vars[<em>key</em>] = <em>value</em></strong>
   </td>
   <td>no shortcut
   </td>
   <td>Update task variables (you must update_task next)
   </td>
  </tr>
  <tr>
   <td><strong>update_task</strong>
   </td>
   <td>u
   </td>
   <td>Recreate a task with updated task variables
   </td>
  </tr>
  <tr>
   <td><strong>redo</strong>
   </td>
   <td>r
   </td>
   <td>Run the task again
   </td>
  </tr>
  <tr>
   <td><strong>continue</strong>
   </td>
   <td>c
   </td>
   <td>Continue executing, starting with the next task
   </td>
  </tr>
  <tr>
   <td><strong>quit</strong>
   </td>
   <td>q
   </td>
   <td>Quit the debugger
   </td>
  </tr>
</table>



     


    _From <[https://docs.ansible.com/ansible/latest/user_guide/playbooks_debugger.html#print-command](https://docs.ansible.com/ansible/latest/user_guide/playbooks_debugger.html#print-command)> _

 


     



* **Demo 1: Configure debugger "always" when YAML data is imported**
* In the below example debugger "always" is enabled for "`Import YAML data for switches`" task.
* The playbook will enter debug mode when executing this task.
* In debug mode we will enter a series of commands to print the variables by accessing the dictionary key/list index.

 

 


```
(venv) root@dev-box:~/ansible-project# cat playbook4_nxos_config_yaml_source.yaml
---
- name: Configure Layer 3 interfaces
  hosts: switches
  gather_facts: no
  tasks: 
    - name: Import YAML data for switches
      ansible.builtin.set_fact:
        interface_data: "{{ lookup('file', 'interface_data.yaml') | from_yaml }}"
      #Enable debugger "always"
      debugger: always
    - name: Configure physical interfaces as Layer 3 interfaces
      cisco.nxos.nxos_config:
        lines: no switchport
        parents: "interface {{ item.interface }}"
      when: item.interface is search("Ethernet")
      #Loop across the devices in YAML file which are keys in Dictionary of YAML file.
      with_items: "{{ interface_data ['%s' | format(inventory_hostname)] }}"
    - name: Configure Layer 3 intefaces on switches
      cisco.nxos.nxos_config:
        lines:
          - "ip address {{ item.ip }}"
          - "description {{ item.description }}"
          - "{{ item.status }}"
        parents: "interface {{ item.interface }}"
      #Loop across the devices in YAML file which are keys in Dictionary of YAML file.
      with_items: "{{ interface_data ['%s' | format(inventory_hostname)] }}"
```



```


(venv) root@dev-box:~/ansible-project# ansible-playbook -i inventory.yaml playbook4_nxos_config_yaml_source.yaml 


PLAY [Configure Layer 3 interfaces] *************************************************************************************************************************************************


TASK [Import YAML data for switches] ************************************************************************************************************************************************
ok: [sw1]
# Print task name.
[sw1] TASK: Import YAML data for switches (debug)> p task
TASK: Import YAML data for switches


# Print hostname that task bring run on
[sw1] TASK: Import YAML data for switches (debug)> p host
sw1


# Print arguments which will print all the variables part of task
[sw1] TASK: Import YAML data for switches (debug)> p task.args
{'interface_data': {'sw1': [{'description': 'Loopback interface for routing '
                                            'protocols',
                             'interface': 'loopback0',
                             'ip': '10.1.1.1/32',
                             'status': 'no shutdown'},
                            {'description': 'To Ethernet1/1 of sw2',
                             'interface': 'Ethernet1/1',
                             'ip': '192.0.2.1/24',
                             'status': 'no shutdown'}],
                    'sw2': [{'description': 'Loopback interface for routing '
                                            'protocols',
                             'interface': 'loopback0',
                             'ip': '10.2.2.2/32',
                             'status': 'no shutdown'},
                            {'description': 'To Ethernet1/1 of sw1',
                             'interface': 'Ethernet1/1',
                             'ip': '192.0.2.2/24',
                             'status': 'no shutdown'},
                            {'description': 'Example SVI',
                             'interface': 'vlan 10',
                             'ip': '192.168.10.1/24',
                             'status': 'no shutdown'}]}}


# Print keys by accessing the dictionary key and list index
[sw1] TASK: Import YAML data for switches (debug)> p task.args["interface_data"]["sw1"][0]
{'description': 'Loopback interface for routing protocols',
 'interface': 'loopback0',
 'ip': '10.1.1.1/32',
 'status': 'no shutdown'}
[sw1] TASK: Import YAML data for switches (debug)> p task.args["interface_data"]["sw1"][0]["ip"]
'10.1.1.1/32'


```


**<code># </code>Continue executing the task, starting with the next task</strong>


```
[sw1] TASK: Import YAML data for switches (debug)> c
ok: [sw2]
[sw2] TASK: Import YAML data for switches (debug)> c


TASK [Configure physical interfaces as Layer 3 interfaces] **************************************************************************************************************************
skipping: [sw1] => (item={'interface': 'loopback0', 'ip': '10.1.1.1/32', 'description': 'Loopback interface for routing protocols', 'status': 'no shutdown'}) 
skipping: [sw2] => (item={'interface': 'loopback0', 'ip': '10.2.2.2/32', 'description': 'Loopback interface for routing protocols', 'status': 'no shutdown'}) 
ok: [sw1] => (item={'interface': 'Ethernet1/1', 'ip': '192.0.2.1/24', 'description': 'To Ethernet1/1 of sw2', 'status': 'no shutdown'})
ok: [sw2] => (item={'interface': 'Ethernet1/1', 'ip': '192.0.2.2/24', 'description': 'To Ethernet1/1 of sw1', 'status': 'no shutdown'})
skipping: [sw2] => (item={'interface': 'vlan 10', 'ip': '192.168.10.1/24', 'description': 'Example SVI', 'status': 'no shutdown'}) 


TASK [Configure Layer 3 intefaces on switches] **************************************************************************************************************************************
changed: [sw2] => (item={'interface': 'loopback0', 'ip': '10.2.2.2/32', 'description': 'Loopback interface for routing protocols', 'status': 'no shutdown'})
changed: [sw1] => (item={'interface': 'loopback0', 'ip': '10.1.1.1/32', 'description': 'Loopback interface for routing protocols', 'status': 'no shutdown'})
ok: [sw2] => (item={'interface': 'Ethernet1/1', 'ip': '192.0.2.2/24', 'description': 'To Ethernet1/1 of sw1', 'status': 'no shutdown'})
ok: [sw1] => (item={'interface': 'Ethernet1/1', 'ip': '192.0.2.1/24', 'description': 'To Ethernet1/1 of sw2', 'status': 'no shutdown'})
[WARNING]: To ensure idempotency and correct diff the input configuration lines should be similar to how they appear if present in the running configuration on device
failed: [sw2] (item={'interface': 'vlan 10', 'ip': '192.168.10.1/24', 'description': 'Example SVI', 'status': 'no shutdown'}) => {"ansible_loop_var": "item", "changed": false, "item": {"description": "Example SVI", "interface": "vlan 10", "ip": "192.168.10.1/24", "status": "no shutdown"}, "msg": "interface vlan 10\r\r\n                       ^\r\nInvalid interface format at '^' marker.\r\n\rSW2(config)# "}


PLAY RECAP **************************************************************************************************************************************************************************
sw1                        : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
sw2                        : ok=2    changed=0    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0 
```


 



* **Demo 2: Configure debugger "on_failed" in playbook task while configuring the interfaces.**
    * In the below example the "interface_data.yaml" is updated to configured VLAN 10 on sw2.
    * However without enabling the feature "interface-vlan" we cannot configure vlan 10 on the switch. Therefore we will get an error when we execute the playbook.
    * Debugger "on_failed" is enabled for "`Configure Layer 3 intefaces on switches`"  task. As the playbook will fail during the task when configuring vlan 10, it will enter debugging.
    * In debug mode we will enter a series of commands to print the variables and change the variables by accessing the dictionary key.

     


<table>
  <tr>
   <td>
<code>(venv) root@dev-box:~/ansible-project# cat interface_data.yaml</code>
<p>
<code>---</code>
<p>
<code>sw1:</code>
<p>
<code>- interface: loopback0</code>
<p>
<code>  ip: 10.1.1.1/32</code>
<p>
<code>  description: Loopback interface for routing protocols</code>
<p>
<code>  status: no shutdown</code>
<p>
<code>- interface: Ethernet1/1</code>
<p>
<code>  ip: 192.0.2.1/24</code>
<p>
<code>  description: To Ethernet1/1 of sw2</code>
<p>
<code>  status: no shutdown</code>
<p>
<code>sw2:</code>
<p>
<code>- interface: loopback0</code>
<p>
<code>  ip: 10.2.2.2/32</code>
<p>
<code>  description: Loopback interface for routing protocols</code>
<p>
<code>  status: no shutdown</code>
<p>
<code>- interface: Ethernet1/1</code>
<p>
<code>  ip: 192.0.2.2/24</code>
<p>
<code>  description: To Ethernet1/1 of sw1</code>
<p>
<code>  status: no shutdown</code>
<p>
<code>- interface: vlan 10</code>
<p>
<code>  ip: 192.168.10.1/24</code>
<p>
<code>  description: Example SVI</code>
<p>
<code>  status: no shutdown</code>
   </td>
   <td><code>(venv) root@dev-box:~/ansible-project# cat playbook4_nxos_config_yaml_source.yaml</code>
<p>
<code>---</code>
<p>
<code>- name: Configure Layer 3 interfaces</code>
<p>
<code>  hosts: switches</code>
<p>
<code>  gather_facts: no</code>
<p>
<code>  tasks: </code>
<p>
<code>    - name: Import YAML data for switches</code>
<p>
<code>      ansible.builtin.set_fact:</code>
<p>
<code>        interface_data: "{{ lookup('file', 'interface_data.yaml') | from_yaml }}"</code>
<p>
<code>      # debugger: always</code>
<p>
<code>    - name: Configure physical interfaces as Layer 3 interfaces</code>
<p>
<code>      cisco.nxos.nxos_config:</code>
<p>
<code>        lines: no switchport</code>
<p>
<code>        parents: "interface {{ item.interface }}"</code>
<p>
<code>      when: item.interface is search("Ethernet")</code>
<p>
<code>      #Loop across the devices in YAML file which are keys in Dictionary of YAML file.</code>
<p>
<code>      with_items: "{{ interface_data ['%s' | format(inventory_hostname)] }}"</code>
<p>
<code>    - name: Configure Layer 3 intefaces on switches</code>
<p>
<code>      cisco.nxos.nxos_config:</code>
<p>
<code>        lines:</code>
<p>
<code>          - "ip address {{ item.ip }}"</code>
<p>
<code>          - "description {{ item.description }}"</code>
<p>
<code>          - "{{ item.status }}"</code>
<p>
<code>        parents: "interface {{ item.interface }}"</code>
<p>
<code>      #Loop across the devices in YAML file which are keys in Dictionary of YAML file.</code>
<p>
<code>      with_items: "{{ interface_data ['%s' | format(inventory_hostname)] }}"</code>
<p>
<strong><code>      #Enable debugger "on_failed"</code></strong>
<code>      debugger: on_failed</code>
   </td>
  </tr>
</table>



     


     


```
    (venv) root@dev-box:~/ansible-project# ansible-playbook -i inventory.yaml playbook4_nxos_config_yaml_source.yaml 


    PLAY [Configure Layer 3 interfaces] ******************************************************************************************************************************************


    TASK [Import YAML data for switches] *****************************************************************************************************************************************
    ok: [sw1]
    ok: [sw2]


    TASK [Configure physical interfaces as Layer 3 interfaces] *******************************************************************************************************************
    skipping: [sw1] => (item={'interface': 'loopback0', 'ip': '10.1.1.1/32', 'description': 'Loopback interface for routing protocols', 'status': 'no shutdown'}) 
    skipping: [sw2] => (item={'interface': 'loopback0', 'ip': '10.2.2.2/32', 'description': 'Loopback interface for routing protocols', 'status': 'no shutdown'}) 
    ok: [sw2] => (item={'interface': 'Ethernet1/1', 'ip': '192.0.2.2/24', 'description': 'To Ethernet1/1 of sw1', 'status': 'no shutdown'})
    skipping: [sw2] => (item={'interface': 'vlan 10', 'ip': '192.168.10.1/24', 'description': 'Example SVI', 'status': 'no shutdown'}) 
    ok: [sw1] => (item={'interface': 'Ethernet1/1', 'ip': '192.0.2.1/24', 'description': 'To Ethernet1/1 of sw2', 'status': 'no shutdown'})


    TASK [Configure Layer 3 intefaces on switches] *******************************************************************************************************************************
    changed: [sw1] => (item={'interface': 'loopback0', 'ip': '10.1.1.1/32', 'description': 'Loopback interface for routing protocols', 'status': 'no shutdown'})
    changed: [sw2] => (item={'interface': 'loopback0', 'ip': '10.2.2.2/32', 'description': 'Loopback interface for routing protocols', 'status': 'no shutdown'})
    ok: [sw1] => (item={'interface': 'Ethernet1/1', 'ip': '192.0.2.1/24', 'description': 'To Ethernet1/1 of sw2', 'status': 'no shutdown'})
    [WARNING]: To ensure idempotency and correct diff the input configuration lines should be similar to how they appear if present in the running configuration on device
    ok: [sw2] => (item={'interface': 'Ethernet1/1', 'ip': '192.0.2.2/24', 'description': 'To Ethernet1/1 of sw1', 'status': 'no shutdown'})
    failed: [sw2] (item={'interface': 'vlan 10', 'ip': '192.168.10.1/24', 'description': 'Example SVI', 'status': 'no shutdown'}) => {"ansible_loop_var": "item", "changed": false, "item": {"description": "Example SVI", "interface": "vlan 10", "ip": "192.168.10.1/24", "status": "no shutdown"}, "msg": "interface vlan 10\r\r\n                       ^\r\nInvalid interface format at '^' marker.\r\n\rSW2(config)# "}


    #Print variables of interface_data.
    [sw2] TASK: Configure Layer 3 intefaces on switches (debug)> p task_vars["interface_data"]
    {'sw1': [{'description': 'Loopback interface for routing protocols',
              'interface': 'loopback0',
              'ip': '10.1.1.1/32',
              'status': 'no shutdown'},
             {'description': 'To Ethernet1/1 of sw2',
              'interface': 'Ethernet1/1',
              'ip': '192.0.2.1/24',
              'status': 'no shutdown'}],
     'sw2': [{'description': 'Loopback interface for routing protocols',
              'interface': 'loopback0',
              'ip': '10.2.2.2/32',
              'status': 'no shutdown'},
             {'description': 'To Ethernet1/1 of sw1',
              'interface': 'Ethernet1/1',
              'ip': '192.0.2.2/24',
              'status': 'no shutdown'},
             {'description': 'Example SVI',
              'interface': 'vlan 10',
              'ip': '192.168.10.1/24',
              'status': 'no shutdown'}]}


    #Print the value by accessing the key.
    [sw2] TASK: Configure Layer 3 intefaces on switches (debug)> p task_vars["interface_data"]["sw2"][2]["interface"]
    'vlan 10'


    #Access the key and change the value to loopback1 from vlan 10
    [sw2] TASK: Configure Layer 3 intefaces on switches (debug)> task_vars["interface_data"]["sw2"][2]["interface"]  = "loopback1"


    #Print the value by accessing the key.
    [sw2] TASK: Configure Layer 3 intefaces on switches (debug)> p task_vars["interface_data"]["sw2"][2]["interface"]
    'loopback1'
```



     


```
    #Rerun the task with update value.
    [sw2] TASK: Configure Layer 3 intefaces on switches (debug)> r   