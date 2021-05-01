## Goal

![DeployNetworkServices Topology](./topology.png)

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

## Prepraration - initiate the topolgy using containerlab