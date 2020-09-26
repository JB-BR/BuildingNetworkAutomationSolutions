- Exectute docker-compose from the dedicated directory : 
```
docker-compose up -d
```

- Get the IP atributed by docker to the container: 

```
sudo docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' YOUR-DIRECTORY_ceos-1_1
```

Example:
```
vagrant@ubuntu-18:/vagrant_data/containers/arista-ceos$ sudo docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' aristaceos_ceos-1_1
172.19.0.2
```

- Start the container bash: 
```
docker exec -it YOUR-DIRECTORY_ceos-1_1 /bin/bash
```


- Configure this IP on the container and ping the docker host
```
bash-4.2# Cli
localhost>enable
localhost#configure terminal
localhost(config)#interface management 0
localhost(config-if-Ma0)#ip address 172.19.0.2/16
localhost(config-if-Ma0)#ping 172.19.0.1
PING 172.19.0.1 (172.19.0.1) 72(100) bytes of data.
80 bytes from 172.19.0.1: icmp_seq=1 ttl=64 time=0.073 ms
80 bytes from 172.19.0.1: icmp_seq=2 ttl=64 time=0.049 ms
80 bytes from 172.19.0.1: icmp_seq=3 ttl=64 time=0.034 ms
80 bytes from 172.19.0.1: icmp_seq=4 ttl=64 time=0.028 ms
80 bytes from 172.19.0.1: icmp_seq=5 ttl=64 time=0.025 ms
```

- Manage cEOS Host with Ansible and the Automator container
[arista.eos.eos_config plugin](https://docs.ansible.com/ansible/latest/collections/arista/eos/eos_config_module.html)
[Automator](https://github.com/JB-BR/BuildingNetworkAutomationSolutions/tree/master/GettingStarted/containers/automator)
[Arista Ansible examples](https://github.com/titom73/ansible-arista-module-howto)
[Arista Ansible examples](https://ansible-arista-howto.readthedocs.io/en/latest/overview.html)
```
docker exec -it automator_automator_1 /bin/bash
cd /vagrant_data/containers/arista-ceos/
# Install ansible galaxy
ansible-galaxy collection install arista.eos
```