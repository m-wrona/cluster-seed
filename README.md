# Cluster

Sample seed for creating cluster for web application.

Cluster is built basing on:
- Nginx: HTTP Server
- Haproxy: Load Balancer
- Docker: virtualization 
- Ansible: deployment & provisioning

# Infrastructure

Basic cluster configuration looks like following:

```preformated
    ┌───────────────────┐                     ┌───────────────────┐
    │ app (node 1)      │                     │ app (node2)       │
    └───────────────┬───┘                     └───┬───────────────┘
                    │                             │
                    │                             │
                    └────────────┐   ┌────────────┘
                                 │   │
                                 │   │
                          ┌──────┴───┴──────┐
                          │ LB (node 1,2)   │
                          └────────┬────────┘
                                   │
                                   │
                                http in
```

Each node is composed of running services of application and LB.

Each service is run on separate Docker container.

# Provisioning

Install all depedendencies and prepare cluster for deployment.

```shell
cd ansible
ansible-playbook -i {{env}} provisioning/site.yml
```

# Deployment

Deploy all application related stuff to the cluster. 

```shell
cd ansible
ansible-playbook -i {{env}} deployment/site.yml
```

# Ansible - common commands

Some samples of handy commands to be familiar with. 

1. Ping

```shell
ansible all -i {{env}} -m ping
```

2. Run a command on all servers

```shell
ansible all -i {{env}} -a "whoami"
ansible all -i {{env}} -a "uptime"
ansible all -i {{env}} -a "date"
ansible all -i {{env}} -a "cat /etc/issue"
ansible all -i {{env}} -a "docker ps"
```

3. Show memory, cpu and other config options on all servers

```shell
ansible all -i {{env}} -m setup
ansible all -i {{env}} -m setup -a "filter=ansible_*_mb"
ansible all -i {{env}} -m setup -a "filter=ansible_processor*"
ansible all -i {{env}} -m setup -a "filter=ansible_all_ipv4_addresses"
ansible all -i {{env}} -m setup -a "filter=ansible_bios_*"
```

5. Run playbook

```shell
ansible-playbook -i {{env}} deployment/site.yml --ask-sudo-pass
ansible-playbook -i {{env}} deployment/site.yml --tags nginx --ask-sudo-pass
```

6. Dry-run, i.e. only check and report what changes should be made without actually executing them

```shell
ansible-playbook -i {{env}} deployment/site.yml --tags "nginx" --ask-sudo-pass --check
ansible-playbook -i {{env}} deployment/site.yml --tags "nginx" --ask-sudo-pass --check --diff
```


