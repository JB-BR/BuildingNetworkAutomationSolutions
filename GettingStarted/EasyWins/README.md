# Easy Wins

## Playbook
I use the arista.eos.eos_command because arista.eos.eos_vlans throws a weird error :
```
fatal: [ceos1]: FAILED! => {"ansible_facts": {"discovered_interpreter_python": "/usr/bin/python"}, "changed": false, "module_stderr": "Traceback (most recent call last):\n  File \"/home/vagrant/.ansible/tmp/ansible-local-125638GwQ8F/ansible-tmp-1603830704.91-12577-128363541597733/AnsiballZ_eos_vlans.py\", line 102, in <module>\n    _ansiballz_main()\n  File \"/home/vagrant/.ansible/tmp/ansible-local-125638GwQ8F/ansible-tmp-1603830704.91-12577-128363541597733/AnsiballZ_eos_vlans.py\", line 94, in _ansiballz_main\n    invoke_module(zipped_mod, temp_path, ANSIBALLZ_PARAMS)\n  File \"/home/vagrant/.ansible/tmp/ansible-local-125638GwQ8F/ansible-tmp-1603830704.91-12577-128363541597733/AnsiballZ_eos_vlans.py\", line 40, in invoke_module\n    runpy.run_module(mod_name='ansible_collections.arista.eos.plugins.modules.eos_vlans', init_globals=None, run_name='__main__', alter_sys=True)\n  File \"/usr/lib/python2.7/runpy.py\", line 188, in run_module\n    fname, loader, pkg_name)\n  File \"/usr/lib/python2.7/runpy.py\", line 82, in _run_module_code\n    mod_name, mod_fname, mod_loader, pkg_name)\n  File \"/usr/lib/python2.7/runpy.py\", line 72, in _run_code\n    exec code in run_globals\n  File \"/tmp/ansible_arista.eos.eos_vlans_payload_ZZQotT/ansible_arista.eos.eos_vlans_payload.zip/ansible_collections/arista/eos/plugins/modules/eos_vlans.py\", line 327, in <module>\n  File \"/tmp/ansible_arista.eos.eos_vlans_payload_ZZQotT/ansible_arista.eos.eos_vlans_payload.zip/ansible_collections/arista/eos/plugins/modules/eos_vlans.py\", line 322, in main\n  File \"/tmp/ansible_arista.eos.eos_vlans_payload_ZZQotT/ansible_arista.eos.eos_vlans_payload.zip/ansible_collections/arista/eos/plugins/module_utils/network/eos/config/vlans/vlans.py\", line 79, in execute_module\n  File \"/tmp/ansible_arista.eos.eos_vlans_payload_ZZQotT/ansible_arista.eos.eos_vlans_payload.zip/ansible_collections/arista/eos/plugins/module_utils/network/eos/config/vlans/vlans.py\", line 46, in get_vlans_facts\n  File \"/tmp/ansible_arista.eos.eos_vlans_payload_ZZQotT/ansible_arista.eos.eos_vlans_payload.zip/ansible_collections/arista/eos/plugins/module_utils/network/eos/facts/facts.py\", line 107, in get_facts\n  File \"/tmp/ansible_arista.eos.eos_vlans_payload_ZZQotT/ansible_arista.eos.eos_vlans_payload.zip/ansible_collections/ansible/netcommon/plugins/module_utils/network/common/facts/facts.py\", line 131, in get_network_resources_facts\n  File \"/tmp/ansible_arista.eos.eos_vlans_payload_ZZQotT/ansible_arista.eos.eos_vlans_payload.zip/ansible_collections/arista/eos/plugins/module_utils/network/eos/facts/vlans/vlans.py\", line 54, in populate_facts\n  File \"/tmp/ansible_arista.eos.eos_vlans_payload_ZZQotT/ansible_arista.eos.eos_vlans_payload.zip/ansible/module_utils/connection.py\", line 185, in __rpc__\nansible.module_utils.connection.ConnectionError: Invalid input (privileged mode required)\n", "module_stdout": "", "msg": "MODULE FAILURE\nSee stdout/stderr for the exact error", "rc": 1}
```

Moreover, Arista CLI / eAPI can output JSON (perfect for programms) by default whereas arista.eos.eos_vlans outpus yaml (good for humans) so why even bother with solving this error?

## Working Notes

###  How Ansible Playbooks Really work
```
           +----------------------------------------+
           |Select hosts to be included in this play|
           +----------------------------------------+
            |                                      |
            v                                      v
    +--------------+                       +--------------+
    | Gather facts |                       | Gather facts |
    +--------------+                       +--------------+
            |                                      |
            v                                      v
    +--------------+                       +--------------+
    |Execute task 1|                       |Execute task 1|
    +--------------+                       +--------------+
            |                                      |
            v                                      v
    +--------------+                       +--------------+
    |Execute task 2|                       |Execute task 2|
    +--------------+                       +--------------+
            |                                      |
            v                                      v
           +----------------------------------------+
           |                Cleanup                 |
           +----------------------------------------+
           
Source: ipspace.net - Ansible online course - Using Ansible - Ansible Playbooks
```

1. Select hosts to be included in this play
The ansible select the hosts to be included in this play
Ansible collects the ansible variables for those hosts as defined on the inventory file or other sources

2. Gather facts
This is the setup module: 
Log in SSH into the device
Copy Python code to the device
Collects facts (data gathered on the hosts) to the ansible variables
This is necessary for linux servers but not supported by most networking vendors

3. Execute task 1


4. Execute task 2

5. Cleanup
Delete the python code on the hosts
Delete all the facts collected during this play

6. Example
Example:
```
---
- hosts: switches                   # Select the hosts in the [switches] catergory of the inventory file
  gather_facts: no                  # Do not gather facts
  tasks:                            # 
    - raw: "show arp"               # Task 1 : Use the SSH raw module to execute a "show arp" on the device
      register: show                #   Register the result of the "show arp" in an Ansible variable called "show"
    - debug: var=show.stdout_lines  # Task 2 : Display the value of the variable "show", which is the result of the show arp command
    - debug: var=vars               # Task 3 : Display all variables available for the device. "vars" is a special Ansible variables containing all variables for the hosts during the play
                                    # Cleanup, the variable "show" is deleted
                                    # Source: ipspace.net - Ansible online course - Using Ansible - Ansible Playbooks
```

### Ideas - Credentials strategy
1. Store clear text credentials in a yaml file outside this repository
2. Use those credentials in ansible to get device credentials from a password vault
3. Regularily rotate the clear text credentials (for example once a month)
4. Rotate the credentials in the password vault even more often