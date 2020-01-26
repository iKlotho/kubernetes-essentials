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

Will use `Flannel` to provide networking between containers. It creates virtual network to allow pods from different
nodes to communicate each other with unique IP.

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



# Lecture: Containers and Pods

* Pods are the smallest building block of the Kube model. It can have one or more containers, a unique IP, storage
  resources. It needed in order to run and manage containers with Kubernetes.

* Create a simple pod running an nginx container:

```bash
cat << EOF | kubectl create -f -
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
EOF
```

* Verify image is running

```bash
kubectl get pods
# describe more information / troubleshooting
kubectl describe pod nginx
# delete 
kubectl delete pod nginx
```

# Lecture: Clustering and Nodes

```bash
# get a list of nodes
kubectl get nodes

# get more information about a spesific node(memory usage eg.)
kubectl describe node $node_name
```


# Lecture: Networking in Kubernetes

Kubernetes creates virtual network across the whole cluster. Every pod on the cluster has a unique IP address to
communicate with other pods. Even if pods are on different nodes.

> We are going to use busybox to access Nginx that in different node from busybox.

* Create 2 nginx pod.

```bash
cat << EOF | kubectl create -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.15.4
        ports:
        - containerPort: 80
EOF
```

* Create a busybox to run curl command

```bash
cat << EOF | kubectl create -f -
apiVersion: v1
kind: Pod
metadata:
  name: busybox
spec:
  containers:
  - name: busybox
    image: radial/busyboxplus:curl
    args:
    - sleep
    - "1000"
EOF

```

* Check IP addresses of pods and try to access nginx.

```bash
kubectl get pods -o wide

# test an nginx pod from different node
kubectl exec busybox -- curl $nginx_pod_ip

# delete deployment
sudo kubectl delete deployment.apps/nginx
```

# Lecture: Kubernetes Architecture and Components

```bash
# Get a list of system pods running in the cluster:
kubectl get pods -n kube-system
```

* etcd: Provides distributes, sync data storage for the cluster state
* kube-apiserver: Kubernetes simple REST API
* kube-controller-manager: Backend for APIs
* kube-scheduler: Determines when to run pods and what node that pods needs to run on.

- kubelet: A service that handles pods and executes containers on each node.  `systemctl status kubelet`
- kube-proxy: Handles networking communication between nodes by adding firewall routing rules.


# Lecture: Kubernetes Deployments

Deployments allow us to automate the management of pods. A deployment allow us to state a desired state and cluster
consistantly works in order to maintain desired state. 


* Scaling:

It helps with scaling up/down. and evenly sprade out multiple pods to allow high availability.

* Rolling updates:

Deployments allow us to rolling updates with incremantal fashion. It will gradually update pods without encouring any
downtime for our users.

* Self-healing:

Event if accidently deleting any pod deployment make sure it meets desired number of pods and will spawn new pods to
replace deleted pods.


> Deployment example

```bash
cat <<EOF | kubectl create -f -
apiVersion: apps/v1
kind: Deployment
metadata:
    name: nginx-deployment
    labels:
        app: ngnix
    spec:
        replicas: 2
        selector:
            matchLabels:
                app: nginx
        template:
            metadata:
                labels:
                    app: nginx
            spec:
                containers:
                -   name: nginx
                    image: nginx:1.15.4
                    ports:
                    -   containerPort: 80

EOF
```


```bash
# Get a list of deployments
kubectl get deployments

# Get more information about a deployment
kubectl describe deployment nginx-deployment

# delete a pod
# because we stated 2 replicas
# after delete it will spawn new pod to match desired state
kubectl delete pod $nginx-deployment-pod-name

Get a list of pods:
kubectl get pods
```


# Lecture: Kubernetes Services

When deploying pods it will constantly change the pods. If some external source try to access deleted port
it will fail. With services it will abstract the list of pods so external source will try to access service
and some of the pods will response to it. So when pods come and go, you get uninterrupted, dynamic access to
whatever replicas are up at the time.

* With previous lectures deployment will add service on top of that deployment pods.

```bash

cat << EOF | kubectl create -f -
kind: Service
apiVersion: v1
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    nodePort: 30080
  type: NodePort
EOF
```

```bash
# get list of services
kubectl get svc

# access to service
# Since this is a NodePort service, we should be able to access it using port 30080 
on any of our cluster's servers. 
curl localhost:30080
```

# Lecture: What are Microservices?


Microservices are small independent services that work together to form a whole application.Microservices provide many
benefits in the design and management of applications, but they also introduce a lot of complexity. Kubernetes can help
manage the additional compolexity that microservices bring.


* Scalability
* Cleaner Code
* Reliability
* Variety of tools
...












