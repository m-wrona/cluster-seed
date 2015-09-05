# Cluster

Sample seed for creating cluster for web application.

Cluster is built basing on:

- Nginx: HTTP Server
- Nginx: Load Balancer (separate instance)
- Docker: virtualization 
- Ansible: deployment & provisioning
- Logstash-forward: monitoring (see monitoring section)

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

Once application is deployed it can be reached via HTTP or HTTPS using cluster address or any node.

# Local development

To speed up local development & testing Vagrant configuration is provided.

In order to star virtual cluster locally just hit:

```shell
vagrant up
```

After that two nodes will be started:

- cluster-node1: 10.10.2.20
- cluster-node2: 10.10.2.21

Don't forget to provide your key later on while performing any activities on cluster, for instance:

```shell
cd ansible
# ansible with provided private key
ansible-playbook -i {{env}} provisioning/site.yml  --private-key=~/.vagrant.d/insecure_private_key
```

# Checking cluster state

After deployment running services on cluster can be checked using following command:

```shell 
ansible all -i {{env}} -a "docker ps "
```

For local cluster on Vagrant it looks like following:

```shell 
ansible all -i local -a "docker ps" --sudo --private-key=~/.vagrant.d/insecure_private_key
```

Sample output of "docker ps" for local cluster:

```preformated
cluster-node1 | success | rc=0 >>
CONTAINER ID        IMAGE               COMMAND                CREATED             STATUS              PORTS                                           NAMES
3f2c06d938d9        nginx               "nginx -g 'daemon of   2 hours ago         Up 23 minutes       0.0.0.0:80->80/tcp, 0.0.0.0:443->443/tcp        cluster-seed-dev-lb-1   
a890a17c89b7        nginx               "nginx -g 'daemon of   2 hours ago         Up 23 minutes       0.0.0.0:10000->80/tcp, 0.0.0.0:10001->443/tcp   cluster-seed-dev-fe-1   

cluster-node2 | success | rc=0 >>
CONTAINER ID        IMAGE               COMMAND                CREATED             STATUS              PORTS                                           NAMES
f72259ba854f        nginx               "nginx -g 'daemon of   2 hours ago         Up 23 minutes       0.0.0.0:80->80/tcp, 0.0.0.0:443->443/tcp        cluster-seed-dev-lb-1   
e0de9527ed1e        nginx               "nginx -g 'daemon of   2 hours ago         Up 23 minutes       0.0.0.0:10000->80/tcp, 0.0.0.0:10001->443/tcp   cluster-seed-dev-fe-1   

```

# Monitoring

Cluster is prepared for monitoring using ELK stack (elasticsearch-logstash-kibana).

Therefore logstash-forward service is installed by default on cluster nodes in order to be able

to send logs to our monitoring cluster. 

Monotoring nodes property in ansible/group_vars/all is defining where logs will be sent: 

```
monitoring_nodes: 
  - "10.10.2.30:{{ logstash_fwd_port }}"
```

Make sure that proper certificate is used on your production environment for logs forwarding.

Like always for local environment sample certificate is provided.

Set-up of monitoring cluster can be checked in repository [cluster monitoring seed](https://github.com/m-wrona/cluster-monitoring-seed)

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


