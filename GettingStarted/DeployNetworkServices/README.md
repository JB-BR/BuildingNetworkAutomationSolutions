## Goal

```
          +-------+                   +-------+
          |csrx-1 +-------------------+csrx-2 |
          +-------+                   +-------+
              |                          |
              |                          |
         +---------+                     |
         |arista-1 |                     |
         +---------+                     |
         |         |                     |
         |         |                     |
    +-------+  +-------+             +-------+
    |client1|  |client2|             |server |
    +-------+  +-------+             +-------+
```

## Todo 
Create an ansible script which generate the topology:
- Create the csrx-1, csrx-2 and arista-1 and publish their SSH Ports
- Create the ansible inventory file for the csrx-1, csrx-2 and arista-1
- Create one docker network bridge per connection
- Attache the bridges to csrx-1, csrx-2 and arista-1
- Create the two client containers and the server and attach them to the correponding briges
- Push the base configuration to csrx-1, csrx-2 and arista-1
- Push the IPSEC Service to csrx-1, csrx-2
- Bonus : Test

## Prepraration - create the vSRX VM
Download the Hyper-V image and follow the instructions to create a base image for vSRX: 
https://www.juniper.net/documentation/us/en/software/vsrx/vsrx-hyper-v/topics/task/security-vsrx-hyper-v-manager-deploying.html

