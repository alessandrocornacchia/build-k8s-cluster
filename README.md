# Create a Kubernetes (k8s) cluster among Virtual Machines with Ansible and Incus

The repository contains scripts to quickly create a `k8s` cluster among virtual machines on a single physical node.
It contains the following main components:
- `create_incus-net.sh`: creates a bridged network within a single node
- `create_vms.sh`: creates a VM
- `ansible/`: contains Ansible playbooks to initialize a Kubernetes cluster using the VMs above

## Requirements
Software packages:
- `incus` 
- `ansible`
Incus:
- image `k8s` in the `local:` image registry

## How to run
There are two dependent steps:  
1. [Create VMs with Incus](./README.md#vm-creation)  
2. [Setup Kubernetes cluster](./README.md#kubernetes-setup-with-ansible)

### VM creation
Login on the node where you want to host the VMs. 
Edit the variable `NET=kashef-albaraa` in the script `create_incus-net.sh`. Then:
```
sudo ./create_incus-net.sh
```

Check if the network has been created successfully and take note of the IP range assigned to it.

```
$ sudo incus network list

+-----------------+----------+---------+-----------------+---------------------------+-------------+---------+---------+
|      NAME       |   TYPE   | MANAGED |      IPV4       |           IPV6            | DESCRIPTION | USED BY |  STATE  |
+-----------------+----------+---------+-----------------+---------------------------+-------------+---------+---------+
+-----------------+----------+---------+-----------------+---------------------------+-------------+---------+---------+
| kashef0         | bridge   | YES     | 10.112.180.1/24 | fd42:52dd:8110:19c7::1/64 |             | 3       | CREATED |
+-----------------+----------+---------+-----------------+---------------------------+-------------+---------+---------+
| kashef-albaraa  | bridge   | YES     | 10.180.100.1/24 | fd42:a79b:5816:4cc::1/64  |             | 3       | CREATED |
+-----------------+----------+---------+-----------------+---------------------------+-------------+---------+---------+
| kashef-cornaca  | bridge   | YES     | 10.178.48.1/24  | fd42:de83:6d94:fe2b::1/64 |             | 3       | CREATED |
+-----------------+----------+---------+-----------------+---------------------------+-------------+---------+---------+
```


Edit the script `create_vms.sh`, specify the name of the VM you want to create and the name of the network it must be connected to (e.g., `kashef-albaraa`).

Then run:
```
sudo ./create_vms.sh
```

**NOTE:** If the image `k8s` is not available in your setup, you can start from a clean `ubuntu` image. However, you need to modify the Ansible scripts to install `kubeadm`, `kubectl`, `docker` and related dependencies. 

Check if the image has been created:
```
sudo incus list
```

### Kubernetes setup with Ansible

Login on the node where you installed your Ansible client. The `ansible/` folder contains automation scripts to initialize and join a multi-node Kubernetes cluster.

#### Create inventory
Create an inventory file with your target machines. Duplicate `inventory-cornaca.yaml` and edit according to the [VMs you created](./README.md#vm-creation).

#### Run playbooks
```
ansible-playbook -K -i <inventory name> <playbook>
```
Use the two Ansible playbooks in the order specified below:
1) `controller.yaml`
2) `workers.yaml`

Replace `<inventory name>` with your new inventory.

#### Advanced usage 
Depending on your needs, you may want to perform some steps manually, but still use Ansible for other tasks. The playbooks contain `tags:` here and there in the code. You can add others.

Then run `ansible-playbook` with the option `-t <tag name>` to filter the subset of tasks you want to execute.

