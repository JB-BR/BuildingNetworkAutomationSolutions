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
Delete all the facts colelcting during this play


## Credentials strategy
1. Store clear text credentials in a yaml file outside this repository
2. Use those credentials in ansible to get device credentials from a password vault
3. Regularily rotate the clear text credentials (for example once a month)
4. Rotate the credentials in the password vault even more often