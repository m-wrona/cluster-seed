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

# Environments 

Due to virtualization all environments can be deployed on single cluster.

Docker will ensure proper sandboxing between environments.

For demo purposes application can be deplyed per following environments:

- local: run on local VMs
- dev: development run on cluster
- prod: production run on cluster

The other approach is to deploy application per version what enables to make canary releases on cluster.

More info about canary releases can be found in [Martin Fowler's article](http://martinfowler.com/bliki/CanaryRelease.html).


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

After deployment running instances can be checked using following command:

```shell 
ansible all -i {{env}} -a "docker ps"
```

Sample output of "docker ps":

```preformated
node1 | success | rc=0 >>
CONTAINER ID        IMAGE               COMMAND                CREATED             STATUS              PORTS                                           NAMES
9a3ee3b0b973        nginx:latest        "nginx -g 'daemon of   50 seconds ago      Up 49 seconds       0.0.0.0:80->80/tcp, 0.0.0.0:443->443/tcp        cluster-seed-dev-lb-1   
b5b5663bea08        nginx:latest        "nginx -g 'daemon of   53 seconds ago      Up 52 seconds       0.0.0.0:10000->80/tcp, 0.0.0.0:10001->443/tcp   cluster-seed-dev-fe-1
```


# Ansible - common commands

Some samples of handy commands to be familiar with. 

1) Ping

```shell
ansible all -i {{env}} -m ping
```

2) Run a command on all servers

```shell
ansible all -i {{env}} -a "whoami"
ansible all -i {{env}} -a "uptime"
ansible all -i {{env}} -a "date"
ansible all -i {{env}} -a "cat /etc/issue"
ansible all -i {{env}} -a "docker ps"
```

3) Show memory, cpu and other config options on all servers

```shell
ansible all -i {{env}} -m setup
ansible all -i {{env}} -m setup -a "filter=ansible_*_mb"
ansible all -i {{env}} -m setup -a "filter=ansible_processor*"
ansible all -i {{env}} -m setup -a "filter=ansible_all_ipv4_addresses"
ansible all -i {{env}} -m setup -a "filter=ansible_bios_*"
```

4) Run playbook

```shell
ansible-playbook -i {{env}} deployment/site.yml --ask-sudo-pass
ansible-playbook -i {{env}} deployment/site.yml --tags nginx --ask-sudo-pass
```

5) Dry-run, i.e. only check and report what changes should be made without actually executing them

```shell
ansible-playbook -i {{env}} deployment/site.yml --tags "nginx" --ask-sudo-pass --check
ansible-playbook -i {{env}} deployment/site.yml --tags "nginx" --ask-sudo-pass --check --diff
```


