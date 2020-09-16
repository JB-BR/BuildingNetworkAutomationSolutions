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

- Docker scenarios are found in [/vagrant_data/containers]{/vagrant_data/containers}. For example, you can start the arista-ceos scenario:
```
cd /vagrant_data/containers/arista-ceos
sudo docker-compose up -d
sudo docker exec -it aristaceos_ceos-1_1 /bin/bash
```

- TODO - Auto configure IP on ceos

- TODO - Install Ansible

- TODO - Manage cEOS Hosts with Ansible

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