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

vagrant@ubuntu-18:~$ pwd
/home/vagrant

vagrant@ubuntu-18:~$ sudo containerlab deploy --topo /vagrant_data/DeployNetworkServices/ipsec_ceos.clab.yaml
INFO[0000] Parsing & checking topology file: ipsec_ceos.clab.yaml
INFO[0000] Creating lab directory: /home/vagrant/clab-IPSEC-VPN
INFO[0000] Creating container: site_b_client
INFO[0000] Creating container: site_a_client
INFO[0000] Creating container: site_a_gateway
INFO[0000] Creating container: site_b_gateway
INFO[0004] Restarting 'site_b_gateway' node
INFO[0004] Restarting 'site_a_gateway' node
INFO[0007] Creating virtual wire: site_a_gateway:eth2 <--> site_b_gateway:eth2
INFO[0007] Creating virtual wire: site_a_gateway:eth1 <--> site_a_client:eth1
INFO[0007] Creating virtual wire: site_b_gateway:eth1 <--> site_b_client:eth1
INFO[0007] Writing /etc/hosts file
+---+-------------------------------+--------------+------------------+-------+-------+---------+----------------+----------------------+
| # |             Name              | Container ID |      Image       | Kind  | Group |  State  |  IPv4 Address  |     IPv6 Address     |
+---+-------------------------------+--------------+------------------+-------+-------+---------+----------------+----------------------+
| 1 | clab-IPSEC-VPN-site_a_client  | 15d563310d61 | alpine:latest    | linux |       | running | 172.20.20.5/24 | 2001:172:20:20::5/64 |
| 2 | clab-IPSEC-VPN-site_a_gateway | fbc5badc3a99 | ceosimage:latest | ceos  |       | running | 172.20.20.2/24 | 2001:172:20:20::2/64 |
| 3 | clab-IPSEC-VPN-site_b_client  | c8825c7c3735 | alpine:latest    | linux |       | running | 172.20.20.4/24 | 2001:172:20:20::4/64 |
| 4 | clab-IPSEC-VPN-site_b_gateway | ed3099e21370 | ceosimage:latest | ceos  |       | running | 172.20.20.3/24 | 2001:172:20:20::3/64 |
+---+-------------------------------+--------------+------------------+-------+-------+---------+----------------+----------------------+


vagrant@ubuntu-18:~$ docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                                         NAMES
fbc5badc3a99        ceosimage:latest    "/sbin/init systemd.…"   15 seconds ago      Up 8 seconds        0.0.0.0:2001->22/tcp, 0.0.0.0:8001->443/tcp   clab-IPSEC-VPN-site_a_gateway
ed3099e21370        ceosimage:latest    "/sbin/init systemd.…"   15 seconds ago      Up 9 seconds        0.0.0.0:2002->22/tcp, 0.0.0.0:8002->443/tcp   clab-IPSEC-VPN-site_b_gateway
c8825c7c3735        alpine:latest       "/bin/sh"                15 seconds ago      Up 12 seconds       0.0.0.0:2004->22/tcp                          clab-IPSEC-VPN-site_b_client
15d563310d61        alpine:latest       "/bin/sh"                15 seconds ago      Up 10 seconds       0.0.0.0:2003->22/tcp                          clab-IPSEC-VPN-site_a_client