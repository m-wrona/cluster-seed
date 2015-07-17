# Cluster

Sample seed for creating cluster for web application.

Cluster is built basing on:
- Nginx: HTTP Server
- Nginx: Load Balancer (separate instance)
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

Docker will ensure proper sand-boxing between environments.

Application can be deplyed per chosen environment, for instance:

- local: run on local VMs (defined for demo purposes)
- dev: development run on cluster (not defined - has to be added)
- prod: production run on cluster (not defined - has to be added)

Take a notice that due to Docker usage, dev and prod can be deployed on the same physical cluster.

In such case they will be run separately as virtual clusters on the same physical machines.

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

# Vagrant

To speed up local testing & usage vagrant configuration is provided.

In order to star virtual cluster locally just run:

```shell
vagrant up
```

Don't forget to provide your key later on while doing activities on cluster, for instance:

```shell
cd ansible
ansible-playbook -i {{env}} provisioning/site.yml  ***--private-key=~/.vagrant.d/insecure_private_key***
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


