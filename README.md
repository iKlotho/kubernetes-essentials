# Kubernetes

- A tool for automated management of containerized applications.
Essentially kubernetes manages containers.

## Orchestration

- Managing multiple servers with containers in it.

- It helps with following topics.
    * High Availability
    * Scaling Resources
    * Deploy new code changes to all containers
    * Add/remove containers

# Lecture: Installing Docker

- Run `install_docker_ub.sh` to build docker

# Lecture: Installing Kubernetes

- Run `install_kubes.sh` to build kubernetes, kubectl, kubeadm and kubelet

    > Kubeadm: Automates building / tearing down  the cluster s
    > Kubelet: Agent between the kubernetes API and docker container. It will automate things
    when to start or tear down components
    > Kubectl: Helps interacting with the cluster

# Lecture: Bootstrapping the Cluster

Will be building a Kube Master and two Kube Nodes.


- Run `bootstrapping_the_master.sh` to setup the Kube Master.

After finishing the build it will output the command for joining Kube Nodes to its master.

When Kube Nodes joined to its Master we can check with `kubectl get nodes` to verify their existence.
The nodes are expected to have a STATUS of NotReady at this point.

```bash
NAME                        STATUS     ROLES    AGE     VERSION
iklotho1c.mylabserver.com   NotReady   master   3m28s   v1.12.7
iklotho2c.mylabserver.com   NotReady   <none>   1s      v1.12.7
iklotho3c.mylabserver.com   NotReady   <none>   17s     v1.12.7
```

# Lecture: Configuring Networking with Flannel

Will use `Flannel` to provide networking between containers.

* Set sysctl value for all servers

```bash
echo "net.bridge.bridge-nf-call-iptables=1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

* Install flannel just for master node 

```bash
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/bc79dd1505b0c8681ece4de4c0d86c5cd2643275/Documentation/kube-flannel.yml
```

* All the nodes should have a STATUS of ready:

```bash
kubectl get nodes
```

* It is also a good idea to verify that the Flannel pods are up and running. Run this command to get a list of system pods

```bash
kubectl get pods -n kube-system
````

> All the backend system pods run in the `kube-system` namespace















