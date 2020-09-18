# Lab with Vagrant
We will use Hyper-V as provider.

- Install / Enable Hyper-V 
- Install Vagrant
- Clone this repository
- Go to https://www.arista.com/en/ and download cEOS-Lab
- Verify the checksum and copy the cEOS tarfile to 
```
cp c:\yourpath\cEOS64-lab-4.24.2.1F.tar.xz c:\yourpath\BuildingNetworkAutomationSolutions\GettingStarted
```
- Edit the *aristaceos* variable in the Vagrantfile to match your cEOS version
- On your machine, create a local "vagrant" user for the SMB share and assign the user to the directory created at the step before
- Start a PowerShell console as "Administrator"
- Change the local directory :
```
cd c:\yourpath\GettingStarted
```
- Start the machine
```
vagrant up --provider=hyperv
```

- If prompted, chose the correct switch
```
    default: Please choose a switch to attach to your Hyper-V instance.
    default: If none of these are appropriate, please open the Hyper-V manager
    default: to create a new virtual switch.
    default:
    default: 1) Default Switch
    default: 2) WSL
    default:
    default: What switch would you like to use? 1
```

- When prompted, enter the SMB credentials created before:
```
    default:
    default: Username (user[@domain]): vagrant
    default: Password (will be hidden):
```

- Log in the machine
```
vagrant ssh
```

- Docker scenarios are found in [/containers](./containers) and are mounted in the /vagrant_data directory via SMB. For example, you can start the arista-ceos scenario:
```
cd /vagrant_data/containers/arista-ceos
docker-compose up -d
docker exec -it aristaceos_ceos-1_1 /bin/bash
```

- Configure the management IP on the ceos container
```
# Get the container IP
docker network inspect oob-automation
# Alternative method - does not give the netmask
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' aristaceos_ceos-1_1
# Configure the IP
docker exec -it aristaceos_ceos-1_1 /bin/bash
Cli
enable
configure terminal
interface management 0
ip address 172.19.0.2/16
ping 172.19.0.1
exit
username admin secret admin privilege 15 role network-admin
management api http-commands
no shutdown
management ssh
username admin
end
write
exit
exit
```

- Create the automator machine ([source](https://packetpushers.net/building-a-docker-network-automation-container/))
```
cd /vagrant_data/containers/automator
docker build -f ./Dockerfile -t automator .
docker image ls
```

- Start the automator container
```
cd /vagrant_data/containers/automator
docker-compose up -d
docker exec -it automator_automator_1 /bin/bash
```

- Ping the ceos container from the automator container
```
root@automator:/projects# ping aristaceos_ceos-1_1
PING aristaceos_ceos-1_1 (172.19.0.2) 56(84) bytes of data.
64 bytes from aristaceos_ceos-1_1.oob-automation (172.19.0.2): icmp_seq=1 ttl=64 time=0.090 ms
64 bytes from aristaceos_ceos-1_1.oob-automation (172.19.0.2): icmp_seq=2 ttl=64 time=0.129 ms
64 bytes from aristaceos_ceos-1_1.oob-automation (172.19.0.2): icmp_seq=3 ttl=64 time=0.190 ms
```

- Manage cEOS Hosts with Ansible
```
docker exec -it automator_automator_1 /bin/bash
cd /vagrant_data/containers/arista-ceos/
ansible all -i inventory.ini -m raw -a "show version" -vvv
```

- Manage cEOS Host with Ansible and REST API - Perform a simple GET
```
docker exec -it automator_automator_1 /bin/bash
cd /vagrant_data/containers/arista-ceos/
ansible-playbook ansible-playbook-uri-get.yaml -i inventory.ini -vvv
```
- Stop the machine
```
vagrant suspend
```

- Restart the machine
```
vagrant resume
```

- Destroy the machine
```
vagrant destroy
```
