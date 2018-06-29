## YAAK (Yet Another Ansible Kubernetes)

Heavily inspired by the work done by @chris-short on his
[rak8s](https://github.com/rak8s/rak8s) project and
[@carlosedp](https://github.com/carlosedp) on his work to port various projects
to the ARM platform.

The goal of this project is to be able to quickly
provision and destroy fully functional Kubernetes clusters using all different
kinds of ARM devices to create fully functioning test and home labs.

Be sure to check out
[arm64-kubernetes-faq](https://github.com/vielmetti/arm64-kubernetes-faq) for
more information about ARM based clusters.

# Prerequisites

## Hardware

* Rock64, Raspberry Pi
* Class 10 SD Cards
* Network connection (wireless or wired) with access to the internet

NOTE: 32-bit support appears to be deprecated for master components in
Kubernetes 1.11, so RPi clusters don't look like they will be supported moving
forward.

## Software

* [Ansible](http://docs.ansible.com/ansible/latest/intro_installation.html) 2.2
  or higher
* [`kubectl`](https://kubernetes.io/docs/tasks/tools/install-kubectl/) should be
  available on the system you intend to use to interact with the Kubernetes
  cluster.
* Ability to SSH into all devices and escalate privileges with sudo

## Stand Up Your Kubernetes Cluster

Once the OS has been flashed onto the ARM device there are a few steps needed in
order to get it working.  If you have DNS set up on your network you can set the
hostname of each node to get name resolution.  This is easier then setting
static IPs for each server.

### Set the hostname

```bash
sudo hostnamectl set-hostname <name>
```

You will also want to update the `/etc/hosts` file to point at the <name> that
the host gets set to. 

### Set a static IP (Raspberry Pi master only)

Edit `/etc/dhcpcd.conf` and add a static entry for the master.

```bash
profile static_eth0
static ip_address=192.168.0.100/24
static routers=192.168.0.1
static domain_name_servers=8.8.8.8
```

Reboot the node just to make sure it gets the correct hostname/static IP.

### Download the latest release or clone the repo

```bash
git clone https://github.com/jmreicha/yaak.git
```

### Modify ansible.cfg and inventory

Modify the `inventory` file to suit your environment. Change the names to your
liking and/or modify the IP addresses.

### Confirm Ansible is working

```bash
ansible -m ping all
```

### Deploy everything

On the first run you can use the defaults.

```bash
ansible-playbook cluster.yml --diff
```

On subsquent deployments you can specify the kube user.

```bash
ansible-playbook cluster.yml --diff -u kube
```

### Deploy the master only

Make sure to update the user in ansible.cfg  to the default user of the
device on the first run, which can be different across devices and OSs.

```bash
ansible-playbook cluster.yml --limit 'kube-master' --diff --check --ask-pass
```

Then on subsequent runs (when authorized_keys has been deployed and ansible.cfg
user has been changed to kube).

```bash
ansible-playbook cluster.yml --limit 'kube-master' --diff
```

Or just append a user

```bash
ansible-playbook cluster.yml --limit 'kube-master' --diff -u kube
```

### Deploy the workers only

Make sure to update the user in ansible.cfg  to the default user of the
device on the first run, which can be different across devices and OSs.

```bash
ansible-playbook cluster.yml --limit 'kube-node-1' --diff --ask-sudo-pass
```

Then on subsequent runs (when authorized_keys has been deployed and ansible.cfg
user has been changed to kube).

```bash
ansible-playbook cluster.yml --limit 'kube-node-1' --diff
```

Or just append a user

```bash
ansible-playbook cluster.yml --limit 'kube-node-1' --diff -u kube
```

### Deploy addons separately

```bash
ansible-playbook cluster.yml --tags 'addons' --diff -u kube
```

### Reset worker nodes

```bash
ansible-playbook reset.yml --diff -u kube
```

## Interacting with Kubernetes

Test your Kubernetes cluster is up and running:

```bash
kubectl get nodes
```

### Adjust settings

All configurable options can be found in `group_vars/all.yml`.

Using these variables you can control things like the Kubernetes version that
gets deployed, which version of Docker to use as well as various other knobs
like whether or not to enable addons.

### Deploy extras

There is a directory named `manifests` that contains additional Kubernetes
components that can be deployed by running the `deploy` script.

These manifests community maintained versions of software to support ARM so
aren't always official.  As support matures, these manifests will be updated.

```bash
./deploy
```

Likewise, these components can be removed using the `teardown` script.

```bash
./teardown
```

## References & Credits

* [rak8s](https://rak8s.io/)
* [kubernetes-arm](https://github.com/carlosedp/kubernetes-arm)
* [prometheus-arm](https://github.com/carlosedp/prometheus-ARM)
* [prometheus-operator-arm](https://github.com/carlosedp/prometheus-operator-ARM)
