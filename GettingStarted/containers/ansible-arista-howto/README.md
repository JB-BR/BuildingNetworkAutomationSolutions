# Arista: Ansible How-To
Source: https://ansible-arista-howto.readthedocs.io/en/latest/INSTALL.html

Note : DNS Resolution Error
Workaround :
```
sudo vi /etc/resolv.conf
nameserver 9.9.9.9
```

Follow the [installation instructions](https://github.com/titom73/ansible-arista-module-howto/blob/master/docs/INSTALL.md)

1. As indicated, install the requirements as well as Arista's "docker-topo" tool:
```
python3 -m pip install virtualenv
python3 -m virtualenv ansible_training
cd ansible_training
source bin/activate
python3 -m pip install git+https://github.com/networkop/docker-topo.git
```

2. Skip the cEOS creation since it is already part of the Vagrant VM Built

3. Verify that you are in the ansible_training directory:
```
vagrant@ubuntu-18:~/ansible_training$ pwd
/home/vagrant/ansible_training
```

4. Clone the ansible-arista-module-howto respository in the ansible_training directory created above, edit the cd
```
# Clone the repository
git clone https://github.com/titom73/ansible-arista-module-howto.git

# Enter repository
cd ansible-arista-module-howto/

# Correct teh cEOS image name
vi ansible-demo-topology.yaml
CEOS_IMAGE: ceosimage:latest

# Create the topology
docker-topo --create ansible-demo-topology.yaml

# Leave the Python venv
deactivate
```

5. Using the cEOS console, verify the container configuration. Note that docker-topo also create the user ansible (with the password "ansible" being encrypted"
```
docker exec -it ansible_ceos2 /bin/bash
Cli
enable
show running-config | include ansible
```

Output:
```
username ansible privilege 15 secret sha512 $6$VIeqppufemK9nLGV$bwenyC/mURjyQdSyL/xEn5GEfGmk1Xl7/iv7vrnUA3Zw/2AMpwS/cdFyKjllVFaYBqEvP.8On1nDk4NSxnoEb0
```

6. You can now start playing with ansible following [those instructions](https://ansible-arista-howto.readthedocs.io/en/latest/EOS_CONFIG.html)
```
cd ansible-content/
ansible-playbook pb.config.lines.simple.yaml --diff
```

Output:
```
PLAY RECAP ****************************************************************
ceos1                      : ok=4    changed=3    unreachable=0    failed=0
ceos2                      : ok=4    changed=3    unreachable=0    failed=0
```

### Playbook analysis
1. In every task, the credentials are passed via the provider "arista_credentials"
```
cat pb.config.lines.simple.yaml
```

Output: 
```
...
  tasks:
    - name: Configure device hostname from lines
      eos_config:
        provider: "{{arista_credentials}}"
...
```

2. "arista_credentials" is imported during the pre_tasks:
```
cat pb.config.lines.simple.yaml
```

Output: 
```
...
- name: Run commands on remote LAB devices
...
  pre_tasks:
    - include_vars: "authentication.yaml"
...
```

3. When opening "authentication.yaml" we can see that it actually uses the standard ansible variables: 
```
cat authentication.yaml
```

Output: 
```
---
arista_credentials:
  transport: 'eapi'
  validate_certs: false
  host: "{{ ansible_host }}"
  username: "{{ ansible_user }}"
  password: "{{ansible_ssh_pass}}"
  timeout: 20
```

4. Which means that username and password can both be found the "inventory.ini" file:
```
cat inventory.ini
```

Output: 
```
[lab]
ceos1 ansible_host=127.0.0.1 ansible_port=8000
ceos2 ansible_host=127.0.0.1 ansible_port=8001

[lab:vars]
ansible_user='ansible'
ansible_ssh_pass='ansible'
```