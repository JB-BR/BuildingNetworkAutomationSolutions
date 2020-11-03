# Easy Wins

## Working Notes

### Preparation
Create two cEOS containers and publish their ports on the docker host. 
Container ceos-1 eAPI is reachable over TCP/8000 (8000->443/tcp)
Container ceos-2 eAPI is reachable over TCP/8001 (8000->443/tcp)
```
vagrant@ubuntu-18:~$ docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                   NAMES
5e722521a952        ceosimage:latest    "/sbin/init systemd.…"   3 weeks ago         Up 10 minutes       0.0.0.0:8001->443/tcp   ceos2
79ce82fb26ac        ceosimage:latest    "/sbin/init systemd.…"   3 weeks ago         Up 33 hours         0.0.0.0:8000->443/tcp   ceos1
```

This minimal configuration is present on both containers:
```
username ansible privilege 15 secret 0 ansible
management api http-commands
   no shutdown
```

The inventory file corresponding to this setup is found in [/vagrant_data/EasyWins/playbooks/inventory.vlans.ini]((./playbooks/inventory.vlans.ini).

Install [Aristas Anisble EOS collection](https://galaxy.ansible.com/arista/eos) as well as [Ansible generel collection](https://galaxy.ansible.com/community/general). We will need the latest for its json_query module [which can manipulate JSON data using JMESPath](https://docs.ansible.com/ansible/latest/user_guide/playbooks_filters.html#selecting-json-data-json-queries).
```
ansible-galaxy collection install arista.eos
ansible-galaxy collection install community.general
```

We also need python3 jmespath in order to work on JSON data with Ansible. 
Source: https://docs.ansible.com/ansible/latest/user_guide/playbooks_filters.html#selecting-json-data-json-queries
Example: https://jmespath.org/tutorial.html
Install JMESPath:
```
sudo apt install python3-jmespath
```

Finally, configure ceos1 and ceos2 with few vlans
```
cd /vagrant_data/EasyWins/playbooks/
ansible-playbook -i inventory.vlans.ini pb.config.ceos1.yaml
ansible-playbook -i inventory.vlans.ini pb.config.ceos2.yaml
```

###  VLAN Compliance playbook
This playbook collects the vlans on each cEOS container and validate that the vlan names matches the naming convention pattern. To execute the playbook, run the following commands

```
cd /vagrant_data/EasyWins/playbooks/
ansible-playbook -i inventory.vlans.ini pb.collect.showvlan.compliance.yaml
```

### Playbook detailed explanation

```
           +------------------------------------------------------------------------------+
           |              1. Select hosts to be included in this play                     |
           +------------------------------------------------------------------------------+
            |                                      |                                    |
            v                                      v                                    |
    +-----------------+                    +-----------------+                          |
    |     Device 1    |                    |     Device 2    |                          |
    +-----------------+                    +-----------------+                          |
            |                                      |                                    |
            v                                      v                                    |
    +-----------------+                    +-----------------+                          |
    | 2. Gather facts |                    | 2. Gather facts |                          |
    +-----------------+                    +-----------------+                          |
            |                                      |                                    |
            v                                      v                                    |
    +-----------------+                    +-----------------+                          |
    |3. Execute task 1|                    |3. Execute task 1|                          |
    +-----------------+                    +-----------------+                          |
            |                                      |                                    |
            :                                      :                                    |
            |                                      |                                    |
            v                                      v                                    |
    +-----------------+                    +-----------------+                          |
    |8. Execute task 6|                    |8. Execute task 6|                          |
    +-----------------+                    +-----------------+                          v
            |                                      |                          +-------------------+
            |                                      |                          |     Localhost     |
            |                                      |                          +-------------------+
            |                                      |                                    |
            |                                      |                                    v
            |                                      |                          +-------------------+
            |                                      |                          | 9. Execute task 7 |
            |                                      |                          +-------------------+
            |                                      |                                    |
            v                                      v                                    v
           +------------------------------------------------------------------------------+
           |                                 10. Cleanup                                  |
           +------------------------------------------------------------------------------+
           
Source: ipspace.net - Ansible online course - Using Ansible - Ansible Playbooks
```


1. Select hosts to be included in this play
Ansible select the hosts to be included in this play:
```
- hosts: lab
```
In this case the hosts under the lab category in the inventory file inventory.vlans.ini will be selected. For all the target hosts, Ansible collects the ansible variables as defined on the inventory file.

2. Gather facts
This step is skipped in the playbook:
```
  gather_facts: false
```
When true, this step is taken care of by the Ansible [setup module](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/setup_module.html). This module: 
* SSH into the remote hosts
* Copies Python code to the remote hosts
* Collects facts on remote hosts using the Python code and store the results in so-called "ansible variables"
Obviously this only works when remote host is a Linux server. For other scenarios (network devices, closed appliances, Web-API, Windows Server) it is not always possible/necessary. 

3. Execute task 1
Execute a "show vlan | json" via the eAPI and save the resust as an Ansible fact in show_vlan. 

Note: 
I use the arista.eos.eos_command because arista.eos.eos_vlans throws a weird error :
```
fatal: [ceos1]: FAILED! => {"ansible_facts": {"discovered_interpreter_python": "/usr/bin/python"}, "changed": false, "module_stderr": "Traceback (most recent call last):\n  File \"/home/vagrant/.ansible/tmp/ansible-local-125638GwQ8F/ansible-tmp-1603830704.91-12577-128363541597733/AnsiballZ_eos_vlans.py\", line 102, in <module>\n    _ansiballz_main()\n  File \"/home/vagrant/.ansible/tmp/ansible-local-125638GwQ8F/ansible-tmp-1603830704.91-12577-128363541597733/AnsiballZ_eos_vlans.py\", line 94, in _ansiballz_main\n    invoke_module(zipped_mod, temp_path, ANSIBALLZ_PARAMS)\n  File \"/home/vagrant/.ansible/tmp/ansible-local-125638GwQ8F/ansible-tmp-1603830704.91-12577-128363541597733/AnsiballZ_eos_vlans.py\", line 40, in invoke_module\n    runpy.run_module(mod_name='ansible_collections.arista.eos.plugins.modules.eos_vlans', init_globals=None, run_name='__main__', alter_sys=True)\n  File \"/usr/lib/python2.7/runpy.py\", line 188, in run_module\n    fname, loader, pkg_name)\n  File \"/usr/lib/python2.7/runpy.py\", line 82, in _run_module_code\n    mod_name, mod_fname, mod_loader, pkg_name)\n  File \"/usr/lib/python2.7/runpy.py\", line 72, in _run_code\n    exec code in run_globals\n  File \"/tmp/ansible_arista.eos.eos_vlans_payload_ZZQotT/ansible_arista.eos.eos_vlans_payload.zip/ansible_collections/arista/eos/plugins/modules/eos_vlans.py\", line 327, in <module>\n  File \"/tmp/ansible_arista.eos.eos_vlans_payload_ZZQotT/ansible_arista.eos.eos_vlans_payload.zip/ansible_collections/arista/eos/plugins/modules/eos_vlans.py\", line 322, in main\n  File \"/tmp/ansible_arista.eos.eos_vlans_payload_ZZQotT/ansible_arista.eos.eos_vlans_payload.zip/ansible_collections/arista/eos/plugins/module_utils/network/eos/config/vlans/vlans.py\", line 79, in execute_module\n  File \"/tmp/ansible_arista.eos.eos_vlans_payload_ZZQotT/ansible_arista.eos.eos_vlans_payload.zip/ansible_collections/arista/eos/plugins/module_utils/network/eos/config/vlans/vlans.py\", line 46, in get_vlans_facts\n  File \"/tmp/ansible_arista.eos.eos_vlans_payload_ZZQotT/ansible_arista.eos.eos_vlans_payload.zip/ansible_collections/arista/eos/plugins/module_utils/network/eos/facts/facts.py\", line 107, in get_facts\n  File \"/tmp/ansible_arista.eos.eos_vlans_payload_ZZQotT/ansible_arista.eos.eos_vlans_payload.zip/ansible_collections/ansible/netcommon/plugins/module_utils/network/common/facts/facts.py\", line 131, in get_network_resources_facts\n  File \"/tmp/ansible_arista.eos.eos_vlans_payload_ZZQotT/ansible_arista.eos.eos_vlans_payload.zip/ansible_collections/arista/eos/plugins/module_utils/network/eos/facts/vlans/vlans.py\", line 54, in populate_facts\n  File \"/tmp/ansible_arista.eos.eos_vlans_payload_ZZQotT/ansible_arista.eos.eos_vlans_payload.zip/ansible/module_utils/connection.py\", line 185, in __rpc__\nansible.module_utils.connection.ConnectionError: Invalid input (privileged mode required)\n", "module_stdout": "", "msg": "MODULE FAILURE\nSee stdout/stderr for the exact error", "rc": 1}
```
Moreover, Arista CLI / eAPI can output JSON by default (perfect for machines) whereas the arista.eos.eos_vlans plugin outpus yaml (good for humans) so why even bother with solving this error?

4. Execute task 2
Displays the content of show_vlan. The "show vlan" output itself is contained within a child element called "vlans" itself contained into the child element "stdout".

5. Execute task 3
Using the json_query module, we create a list of vlan names by selecting the "name" element in all vlans. This list is stored into the Ansible fact "vlan_list".

6. Execute task 4
Display the list of vlan names stored in "vlan_list".

7. Execute task 5
Initialize the Ansible facts for the report with their default values. By default the compliance test is passed ("report_status: true").

8. Execute task 6
Loop through the "vlan_list". If a vlan name does not match the naming convention defined by the [regex](https://regex101.com/r/5qdM4N/1), the "report_status" is set to false and the faulty vlan name is added to the "report_reason" list. 

9. Execute task 7
The report is a file containing a JSON objects containing all compliance test results. 
For example, for the Arista device n : 
- hosts[n] : the device hostname
- status[n] : the device compliance status (either true or false)
- reason[n] : an array containing the device non-compliant VLANs (if any)
This report should be consumed by a monitoring tool which will raise an alert when any of the devices status is "false".

10. Cleanup
Delete all the facts collected during this play
When managing linux servers, piece of Python code uploaded to the targets are also deleted. 
