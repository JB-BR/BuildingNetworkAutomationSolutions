# Easy Wins

## How Ansible Playbooks Really work
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

## Credentials strategy
1. Store clear text credentials in a yaml file outside this repository
2. Use those credentials in ansible to get device credentials from a password vault
3. Regularily rotate the clear text credentials (for example once a month)
4. Rotate the credentials in the password vault even more often