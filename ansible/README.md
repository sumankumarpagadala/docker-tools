## Ansible for Kubernetes and Docker Swarm

This directory contains playbooks to set up Kubernetes or Docker swarm
nodes in a secure way, by techniques recommended in Kubernetes and
Docker documentation plus numerous 3rd-party best-practices
recommendations. It also has tools to configure LVM for LUKS encrypted
disk volumes.

Ansible is only used here to install and manage cluster nodes, not to
manage containerized applications. Those can be managed by directly
modifying a SCM-managed set of flat-text configuration files, with
Makefiles to redeploy containers. This greatly reduces maintenance
effort.

### Usage
* Generate your Docker API and registry SSL certs, putting the certs
  (not keys) into roles/docker_node/files/certs. You can optionally use
  a self-signed API cert by storing the same cert as ca-root.pem.
* Define variables as listed below, in your group_vars directory.
* Define host inventory as cluster nodes and managers, in hosts. For
  kubernetes, use group names k8s_master and k8s_nodes; for swarm,
  use group swarm_managers and swarm_nodes.
* Add any ssh public keys for your user(s) into files/keys/ssh_public_keys,
  with file names <user>-<keyname>.pub; these will be added to
  ~<user>/.ssh/authorized_keys.

This includes a Makefile with the following targets, to be run in the
following (approximate) order for kubernetes (see detailed readme in k8s
directory):
```
make k8s-master
cd ../k8s ; make install
make k8s-node
```

For swarm, do this:
```
make test
make volume-setup
make node-setup join-swarm
```

### Variables
Local variables:

Item|Default|Notes
----|-------|-----
local_group_additions| |Unix groups to install
local_users| |Unix users to install
masterkey| |Definitions for LUKS keyserver
network_mode|dhcp|Set primary interface to static or dhcp
nagios_override.allowed_hosts|localhost|Your monitoring server(s)
syslog.host|localhost|Central syslog server
syslog.port|514|Port for syslog
network_mounts| |Network mount points
volumes| |Encrypted LUKS volumes to install

Look in the defaults/main.yml files within each role for the full
list of variables that can be overridden.

Ansible vault variables:

Item|Notes
----|-----
vault_cert_keys.docker_tls|TLS certificate key for Docker API
vault_cert_keys.registry|SSL certificate key for Docker Registry
vault_k8s.join_token|cluster token, generate with kubeadm token create
vault_k8s.encryption_key|used for etcd and secrets encryption
vault_ntp.key|Your ntp peering key
vault_ssh_keys_private.masterkey|Private ssh key for LUKS key server
vault_users|Hash containing hashed user 'password' and 'google_token' values

### Tags

These tags can be used with --skip-tags to disable features (at the
role level) when invoking the main node-setup.yml playbook:

Item|Notes
----|-----
network|Network setup
nagios|Sets up monitoring agent
ntp|Configures NTP

### Extras

#### mythfrontend-setup
This is a companion to the mythtv-backend image: an ansible playbook to
set up MythTV front-ends after PXE booting them with Ubuntu (server) image.
