<p align="center"><img width="150" alt="portfolio_view" src="logo.png"></p>
<!-- <h4 align="center"><a href="HYPERLINK HERE">URL HERE</a></h3> -->
<h1 align="center">Certified Kubernetes Administrator Notes</h1>

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [Exam Outline](#exam-outline)
  - [Software](#software)
  - [Resources](#resources)
- [Basics](#basics)
- [Architecture](#architecture)
  - [Control Plane](#control-plane)
  - [Nodes](#nodes)
  - [Kubernetes CLI tools](#kubernetes-cli-tools)
- [Installation](#installation)
  - [`containerd` Installation (if needed)](#containerd-installation-if-needed)
  - [Kubernetes Installation](#kubernetes-installation)
- [Setup / Initialization](#setup-initialization)
- [Namespaces (ns)](#namespaces-ns)
  - [Working with Namespaces](#working-with-namespaces)
- [Cluster Management](#cluster-management)
  - [High Availability](#high-availability)
  - [Management Tools](#management-tools)
  - [Safely Draining a Node](#safely-draining-a-node)
  - [Upgrading Kubernetes and Tools](#upgrading-kubernetes-and-tools)
    - [Control Plane Node](#control-plane-node)
    - [Worker Node](#worker-node)
  - [Backing up and Restoring `etcd` Cluster Data](#backing-up-and-restoring-etcd-cluster-data)
- [Object Management](#object-management)
  - [Declarative vs. Imperative Methods](#declarative-vs-imperative-methods)
- [Role-Based Access Control (RBAC) Authorization](#role-based-access-control-rbac-authorization)
  - [Role Objects](#role-objects)
    - [Role](#role)
    - [ClusterRole](#clusterrole)
    - [RoleBinding](#rolebinding)
    - [ClusterRoleBinding](#clusterrolebinding)
  - [User Permissions](#user-permissions)
    - [Working With Permissions](#working-with-permissions)
  - [Service Accounts (sa)](#service-accounts-sa)
  - [Inspecting Resource Usage](#inspecting-resource-usage)
- [Pods and Containers](#pods-and-containers)
  - [Application Configuration](#application-configuration)
    - [ConfigMaps (cm)](#configmaps-cm)
    - [Secrets](#secrets)
  - [Managing Container Resources](#managing-container-resources)
  - [Monitoring Container Health with Probes](#monitoring-container-health-with-probes)
    - [Liveness Probe](#liveness-probe)
    - [Startup Probe](#startup-probe)
    - [Readiness Probe](#readiness-probe)
  - [Self-Healing Pods with Restart Policies](#self-healing-pods-with-restart-policies)
  - [Multi-Container Pods](#multi-container-pods)
  - [Init Containers](#init-containers)
- [Scheduling in Kubernetes](#scheduling-in-kubernetes)
  - [Pod Scheduling Configurations](#pod-scheduling-configurations)
    - [`nodeSelector`](#nodeselector)
    - [`nodeName`](#nodename)
  - [DaemonSets (ds)](#daemonsets-ds)
  - [Static Pods](#static-pods)
- [Deployments](#deployments)
  - [Scaling Applications with Deployments](#scaling-applications-with-deployments)
  - [Rolling Updates with Deployments](#rolling-updates-with-deployments)
- [Networking](#networking)
  - [Network Model](#network-model)
  - [Container Network Interface (CNI) Plugins](#container-network-interface-cni-plugins)
  - [Domain Name System (DNS) in Kubernetes](#domain-name-system-dns-in-kubernetes)
  - [NetworkPolicies (netpol)](#networkpolicies-netpol)
  - [Troubleshooting Network](#troubleshooting-network)
- [Services](#services)
  - [Types of Services](#types-of-services)
    - [ClusterIP (default)](#clusterip-default)
    - [NodePort](#nodeport)
    - [LoadBalancer](#loadbalancer)
    - [ExternalName](#externalname)
  - [Service DNS Names](#service-dns-names)
  - [Ingress (ing)](#ingress-ing)
    - [Ingress Controllers](#ingress-controllers)
- [Storage](#storage)
  - [Volumes](#volumes)
  - [Volume Types](#volume-types)
    - [Common Volume Types](#common-volume-types)
  - [Persistent Volumes](#persistent-volumes)
    - [PersistantVolume (pv)](#persistantvolume-pv)
    - [StorageClass (sc)](#storageclass-sc)
    - [PersistentVolumeClaim (pvc)](#persistentvolumeclaim-pvc)
- [Troubleshooting](#troubleshooting)
  - [Kubernetes API server down](#kubernetes-api-server-down)
  - [Node is having problems](#node-is-having-problems)
  - [Cluster System Pod is having problems](#cluster-system-pod-is-having-problems)
  - [Kubernetes Service Logs](#kubernetes-service-logs)
  - [Cluster Component Logs](#cluster-component-logs)
  - [Application Issues](#application-issues)
  - [Networking Issues](#networking-issues)
    - [`netshoot` Tool](#netshoot-tool)
- [Tips and Tricks](#tips-and-tricks)
- [Random](#random)

<!-- /code_chunk_output -->


# Exam Outline

## Software

- Kubernetes version: 1.23
- Ubuntu 20.04
- Terminal
- Tools available
    - VIM - Text/Code editor
    - tmux - Terminal multiplexor
    - jq - Working with JSON format
    - yq - Working with YAML format

## Resources

- Kubernetes Documentation


# Basics

- Kubernetes (k8s)
    - Definition: Portable, extensible, open-source platform for managing containerized workloads and services, that facilitates both declarative configuration and automation.
    - Container orchestration
    - Application reliability (self-healing)
    - Automation



# Architecture

## Control Plane

- Manages the cluster
- Components
    - **kube-api-server**
       - Serves Kubernetes API
       - Primary interface to control plane and cluster itself
    - **etcd**
        - Backend data store for Kubernetes cluster
    - **kube-scheduler**
        - Selects an available node in the cluster on which to run containers
    - **kube-controller-manager**
        - Runs collection of multiple controller utilities in a single process
    - **cloud-controller-manager**
        - Interface between Kubernetes and various cloud platforms
        - Only used when using with cloud-based resources (i.e. GCP, AWS, Azure)

## Nodes

- Machines where the containers managed by the cluster are run
- Components
    - **kubelet**
        - Kubernetes agent that runs on a node
        - Communicates with control plane
        - Handles reporting /container status and other data to control plane
    - **Container runtime**
        - Not build into Kubernetes. Separate software.
        - Responsible for running containers on machine
        - Kubernetes supports multiple, i.e. Docker, containerd, etc.
    - **kube-proxy**
        - Network proxy that runs on each node
        - Provides network between containers and cluster


## Kubernetes CLI tools

- `kubeadm`
    - Tool that will simply the process of setting up our Kubernetes cluster
    - Can be used to set up multi-node Kubernetes cluster
- `kubectl`
    - Controls the Kubernetes cluster
    - Communicates with the control plane
- `kubelet` 
    - Primary node agent that runs on each Kubernetes node
    - Can register node with the API server
    - Applies, creates, updates, and destroys containers on a node
    - CAN BE USED BY ITSELF AS CLI TOOL??
- More ...
    - HYPERLINK ???


# Installation

The following is a way of installing and setting up Kubernetes. However, you can see other
methods here. The different method depend on how and where Kubernetes is set up.
    - https://itnext.io/kubernetes-installation-methods-the-complete-guide-1036c860a2b3

## `containerd` Installation (if needed)

Perform these steps on both control and worker nodes. The following is on a Debian-based system.

- Load needed kernel modules (ensures when system starts up, modules will be enables)
    - ```bash
      cat << EOF | sudo tee /etc/modules-load.d/containerd.conf
      overlay
      br-netfilter
      EOF
      ```
- Enable the kernel modules right now
    - `sudo modprobe overlay`
    - `sudo modprobe br-netfilter`
- System level networking settings
    - ```bash
        cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
            net.bridge.bridge-nf-call-iptables = 1
            net.ipv4.ip_forward = 1
            net.bridge.bridge-nf-call-ip6tables = 1
        EOF
      ```

- Command to make networking settings take effect immediately
   - `sudo sysctl --system`

- Install **containerd**
    - `sudo apt-get update`
    - `sudo apt-get install -y containerd`

- Create containerd configuration directory
    - `sudo mkdir -p /etc/containerd`

- Set default configurations
    - `sudo containerd config default | sudo tee /etc/containerd/config.toml`

- Restart containerd service
    - `sudo systemctl restart containerd`


## Kubernetes Installation

Perform these steps on both control and worker nodes.

- Disable system swap memory. WHY???
    - `sudo swapoff -a`

- Check system `fstab` file for any entries that will turn swap back on
    - `sudo cat /etc/fstab`

- Install dependencies
    - `sudo apt-get update`
    - `sudo apt-get install -y apt-transport-https ca-certificates curl`

- Add Google Cloud public signing key to `apt` for the Kubernetes repository
    - `curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -`

- Setup the Kubernetes repository entry in `apt`
    - ```bash
        cat << EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
        deb https://apt.kubernetes.io/ kubernetes-xenial main
        EOF
      ```

- Fetch newly added repository information
    - `sudo apt-get update`

- Install Kubernetes tools `(note the version)`
    - List of Kubernetes versions: https://kubernetes.io/releases/
    - `sudo apt-get install -y kubelet=1.23.0-00 kubeadm=1.23.0-00 kubectl=1.23.0-00`

- Prevent automatic updating of Kubernetes packages for more control
    - `sudo apt-mark hold kubelet kubeadm kubectl`


# Setup / Initialization

Perform these steps on the control node.

- Initialize the cluster
    - `sudo kubeadm init --pod-network-cidr 192.168.0.0/16 --kubernetes-version 1.23.0`
    - Running this will provide a set of commands for further setup

- From the previous command output, run the following
    - `mkdir -p $HOME/.kube`
    - `sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config`
    - `sudo chown $(id -u):$(id -g) $HOME/.kube/config`

- Verify clusters is working
    - `kubectl get nodes`

A this point the cluster is running, however it is in a `NotReady` status, because networking
has not been configured yet. For this we need a seperate networking plugin.

- Setting up Calico networking and network security plugin via a remote YAML file
    - https://projectcalico.docs.tigera.io/about/about-calico
    - `kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml`

- Get a node join command to use on nodes to add to cluster
    - `kubeadm token create --print-join-command`
    - Copy the resulting output

Run the following command on each Kubernetes node. Make sure to run as `sudo`

- `sudo kubeadm join <REST OF COMMAND>`



# Namespaces (ns)

Docs: https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/

- Mechanism for isolating groups of resources (i.e. pods, containers) within a single cluster
- Names of resources need to be unique within a namespace, but not across namespaces.
- A way to divide cluster resources between multiple users
- Virtual clusters backed by same physical cluster (similar to Virtual private networks)
- All clusters have the `default` namespace
- `kube-system` namespace is for system components
- Some resources like nodes and persistantVolumes are not in any namespace


## Working with Namespaces

- Listing existing namespaces
    - `kubectl get namespaces`

- Can specify where the command can run with `--namespace`
    - Example: `kubectl get pods --namespace my-namespace`

- Can specify to look at all namespaces with `--all-namespaces` (or `-A`)
    - Can be slow to complete
    - Example `kubectl get pods --all-namespaces`

- Create a custom namespace
    - `kubectl create namespace my-namespace`

- Set a default namespace for all subsequent commands
    - `kubectl config set-context --current --namespace=<NAMESPACE-NAME>`




# Cluster Management

## High Availability

- Need multiple control plane nodes
    - Each control plane node has instance of `kube-api-server`

- Design Patterns
    - Load Balancer
        - All control plane nodes communicate with a load balancer
        - Load balancer communicates with worker nodes with `kubelet`

    - Stacked `etcd`
        - `etcd` runs on the same nodes as control plane components
        - Control planes have its own `etcd` instance
        - Clusters that are set up using `kubeadm` use this design pattern

    - External `etcd`
        - `etcd` runs on complete separate nodes apart from the control plane
        - Can have any number of control plane instances, and any number of `etcd` nodes



## Management Tools

Interface and make it easier to setup or use Kubernetes.

- `kubectl`
    - Main method for Kubernetes

- `kubeadm`
    - Quickly and easily install and setup Kubernetes cluster

- `minicube`
    - Allows to automatically set up local single-node Kubernetes cluster
    - Great for quick development purposes
    - Can add nodes
    - Quickly start and stop a cluster

- `kind`
    - Run local Kubernetes cluster using Docker
    - Can be used for local cluster testing

- `helm`
    - Templating and package management for Kubernetes objects
    - Manage your own templates (known as charts)
    - Can download and share templates

- `Kompose`
    - Translate Docker compose files into Kubernetes objects
    - Allows porting from Docker compose to Kubernetes

- `Kustomize`
    - Configuration management tool for Kubernetes object configurations
    - Share and re-use templated configuration for Kubernetes



## Safely Draining a Node

When performing maintenance, you may sometimes need to remove a Kubernetes node from service/cluster.
This allows all applications on the cluster to run without any interruptions.
Containers will gracefully terminated.

These commands are run on the control plane node.

- Draining a node
    - `kubectl drain <NODE NAME>`
    - `--igrnoe-daemonsets` - Ignore DaemeonSets pods tied to node
    - `--force` - Ignore error messages such as DaemonSet-managed pods

- After maintenance is complete, allow pods to run on node again
    - `kubectl uncordon <NODE NAME>`



## Upgrading Kubernetes and Tools

### Control Plane Node

1. Drain node, putting it out of cluster service
    - `kubectl drain <CONTROL PLANE NODE NAME> --ignore-daemonsets`

2. Upgrade `kubeadm` CLI tool
    - `sudo apt-get update`
    - `sudo apt-get install -y --allow-change-held-packages kubeadm=1.22.2-00`

3. Plan the cluster upgrade
    - `sudo kubeadm upgrade plan v1.22.2`
    - `sudo kubeadm upgrade plan` 
        - Shows versions you can upgrade to
        - Shows any component configs that require manual upgrade
        - Automatically renews certificates that it manages on this node

4. Apply the cluster upgrade
    - `sudo kubeadm upgrade apply -y v1.22.2`

5. Upgrade `kubelet` and `kubectl` CLI tools
    - `sudo apt-get install -y --allow-change-held-packages kubelet=1.22.2-00 kubectl=1.22.2-00`
    - `sudo systemctl daemon-reload`
    - `sudo systemctl restart kubelet`

6. Uncordon the node, putting it back into cluster service
    - `kubectl uncordon <CONTROL PLANE NODE NAME`


### Worker Node

1. Drain node, putting it out of cluster services
    - **This command is run on the control plane node**
    - `kubectl drain <WORKER NODE NAME> --ignore-daemonsets`
    - Same as the control plane node command for draining
    - May have to use `--force`

2. Upgrade `kubeadm` CLI tool
    - `sudo apt-get update`
    - `sudo apt-get install -y --allow-change-held-packages kubeadm=1.22.2-00`

3. Upgrade the `kubelet` configuration
    - `sudo kubeadm upgrade node`

4. Upgrade `kubelet` and `kubectl` CLI tools

5. Uncordon the node, putting it back into cluster service
    - **This command is run on the control plane node**
    - `kubectl uncordon <WORKER PLANE NODE NAME`



## Backing up and Restoring `etcd` Cluster Data

`etcd` is the backend key-value data storage solution for Kubernetes cluster.
All Kubernetes objects, applications, and configurations are stored in etcd.

- etcd website: https://etcd.io/

- Need to use `etcdctl` CLI tool
    - Select API version with `ETCDCTL_API` environmental variable
    - `etcdctl get <KEY>` - Get a specific value for a key

- etcd typically runs on port `2379`

- If TLS encryption, must specify certificates
    - Example: Looking up value for `cluster.name` key
        - ```bash
            ETCDCTL_API=3 etcdctl get cluster.name \
              --endpoints=https://10.0.1.101:2379 \
              --cacert=/home/cloud_user/etcd-certs/etcd-ca.pem \
              --cert=/home/cloud_user/etcd-certs/etcd-server.crt \
              --key=/home/cloud_user/etcd-certs/etcd-server.key
          ```

- Back up data
    - Docs: https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster
    - `ETCDCTL_API=3 etcdctl --endpoints $ENDPOINT snapshot save <FILE NAME>`
        - Here `$ENDPOINT` is a list of host addresses for etcd servers
    - Stop etcd
        - `sudo systemctl stop etcd`
    - Remove existing etcd data
        - `sudo rm -rf /var/lib/etcd`

- Restore the data
    - Docs: https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#restoring-an-etcd-cluster
    - Creates a new logical cluster
    - `ETCDCTL_API=3 etcdctl snapshot restore <FILE NAME>`
    - Example:
        - `ETCDCTL_API=3 etcdctl --endpoints 10.2.0.9:2379 snapshot restore snapshotdb`
    - Set ownership
        - `sudo chown -R etcd:etcd /var/lib/etcd`
    - Start etcd
        - `sudo systemctl start etcd`





# Object Management

Object management is done with `kubectl` CLI tool used to deploy applications, inspect
and manage cluster resources, and view logs.

Docs: https://kubernetes.io/docs/reference/kubectl/

- Usage: `kubectl [COMMAND] [OBJECT TYPE] [OBJECT NAME] [FLAGS]`

- **`kubectl api-resources`**
    - List all available Kubernetes resource objects for current Kubernetes version.

- **`kubectl get ....`**
    - Get a list of available specified objects
    - Docs: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
    - Some options:
        - `-o, --output` - Specify output format (i.e. `json`, `yaml`, `wide`, etc)
        - `--sort-by` - Sort output using a JSONPath expression (i.e. `{.metadata.name}`)
            - Useful to first output result in `json` to see what the JSONPath is
        - `-l, --selector` - Filter results by label (i.e. `key1=value1,key2=value2`)
    - Examples:
        - `kubectl get pods`
        - `kubectl get pv -o wide`
        - `kubectl get pods my-pod -o yaml > my-pod.yml`

- **`kubectl describe ....`**
    - Show detailed and human readable information of a specific resource
    - Docs: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#describe
    - Examples:
        - `kubectl describe pods/nginx`
        - `kubectl describe -f pod.json` - Pod defined by `pod.json`

- **`kubectl create ....`**
    - Create a Kubernetes resource from file or stdin
    - Docs: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#create
    - If object exists, error occurs
    - Can use `--dry-run=clinet`
    - Examples:
        - `kubectl create -f pod.yaml`
        - `cat pod.json | kubectl create --filename -`
        - `kubectl create deployment my-dep --image alpine --dry-run=client -o yaml`

- **`kubectl apply ....`**
    - Will create a new object or modify an existing one
    - Docs: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
    - Will not throw error if object exists
    - Examples:
        - `kubectl apply -f pod.yaml`
        - `kubectl apply -k my-dir/` - Will look into `my-dir` directory for all resource configs

- **`kubectl delete ....`**
    - Delete resources from cluster
    - Docs: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#delete
    - Can specify objects to delete form:
        - Name
        - stdin
        - resources and names (i.e. `pod myPod1 myPod2`)
        - resources and label selector (i.e. `-l name=myLabel`)
    - Some options:
        - `--all` - Delete all specified resources in current namespace
        - `--now` - Immediate shutdown, minimal delay
        - `--force` - Bypass graceful deletion
    - Examples:
        - `kubectl delete -f ./pod.yaml`
        - `kubectl delete pod some-pod --now`

- **`kubectl exec ....`**
    - Execute a command in a container
    - Docs: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#exec
    - Great for troubleshooting
    - Raw terminal interactive mode inside container:
        - `kubectl exec <POD NAME> -c <CONTAINER> -i -t -- bash`
    - Examples:
        - `kubectl exec some-pod -- echo "yo"` - Using first container in pod
        - `kubectl exec some-pod -c python-container -- printenv`
        - `kubectl exec deploy/myDeployment -- date` - Using first pod, first container, in deployment
        - `kubectl exec svc/myService -- ls /dir` - Using first pod, first container, in service


## Declarative vs. Imperative Methods

- Declarative
    - Define objects using data structures such as YAML or JSON (predefined)
    - Example:
        - `kubectl apply -f deployment.yml`

- Imperative
    - Define objects using `kubectl` commands and flags.
    - Some people find imperative commands faster.
    - Can be faster in the exam sometimes!
    - Example:
        - `kubectl create deployment myi-deploytment image=nginx`



# Role-Based Access Control (RBAC) Authorization

Role-based access control (RBAC) is a method of regulating access to resources based on
the roles of individual users within the organization. Essentially, what users are allowed
to do and access within the cluster.

Docs: https://kubernetes.io/docs/reference/access-authn-authz/rbac/


## Role Objects

### Role

- Sets permissions within a particular namespace
- Can specify `namespace`
- Example: Role definition YAML file
    - ```yaml
        apiVersion: rbac.authorization.k8s.io/v1
        kind: Role
        metadata:
          namespace: default    # <-- Note namespace defined
          name: pod-reader
        rules:
        - apiGroups: [""]       # <-- "" indicates the core API group
          resources: ["pods", "pods/logs"]
          verbs: ["get", "watch", "list"]
      ```


### ClusterRole

- Not namespace specific, cluster-wide
- Do not need to specify `namespace`


### RoleBinding

- Grants permission defined in a role to a user or set of users
- Holds a list of subjects (users, groups, or service accounts)
- Holds reference to the role being granted
- Can only reference an Role in the same namespace
- Can also reference a ClusterRole
- Example:
    - ```yaml
        apiVersion: rbac.authorization.k8s.io/v1
        kind: RoleBinding
        metadata:
          name: read-pods       # <-- Has to match the Role metadata.name
          namespace: default
        subjects:               # <-- You can specify more than one "subject"
        - kind: User
          name: jane            # <-- "name" is case sensitive
          apiGroup: rbac.authorization.k8s.io
        roleRef:                # <-- "roleRef" specifies the binding to Role/ClusterRole
          kind: Role            # <-- This must be Role/ClusterRole
          name: pod-reader      # <-- This must match the name of Role/ClusterRole
          apiGroup: rbac.authorization.k8s.io
        ```


### ClusterRoleBinding

- To bind a ClusterRole to all namespaces, use ClusterRoleBinding




## User Permissions


### Working With Permissions

- Get all roles defined in a specific namespace
    - `kubectl get role --namespace <NAMESPACE>`

- Apply a role or role binding definition
    - `kubectl apply -f <ROLE FILE>`

- Check if user has certain access
    - `kubectl get pods --namespace <NAMESPACE> --kubeconfig <USER CONFIG>`

- Check permissions as current user

    - Example: Check to see if I can do everything in my current namespace ("*" means all)
        - `kubectl auth can-i '*' '*'`

    - Example: Check to see if I can create pods in any namespace
        - `kubectl auth can-i create pods --all-namespaces`

    - Example: Check to see if I can list deployments in my current namespace
        - `kubectl auth can-i list deployments.extensions`

- Check permissions as someone else

    - Example: Check if `john.blah` can create deployments
        - `kubectl auth can-i create deployments --namespace default --as john.blah`



## Service Accounts (sa)

Service account provides an identity for processes that run in a Pod.
Users are authenticated to Kubernetes API with User Accounts, but processes in containers inside Pods
are authenticated a Service Accounts.

- Service accounts exist within namespaces
- Can use RBAC objects to control service accounts
- Can bind service accounts with ClusterRoles or ClusterRoleBindings to provide access

- Get/list service accounts
    - `kubectl get sa`

- Creating service accounts
    - `kubectl create -f <SERVICE ACCOUNT CONFIG FILE>`
    - `kubectl create sa <NAME> -n <NAMESPACE>`
- Example: Service account definition
    - ```yaml
        apiVersion: v1
        kind: ServiceAccount
        metadata:
          name: build-robot
        automountServiceAccountToken: false
      ```


## Inspecting Resource Usage

Kubernetes Metrics Server
- Need add-on to collect and provide metrics data about resources, pods, and containers
- Need Metrics API for this
    - Install: `kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml`
- Once a metrics add-on is installed, can use `kubectl top` to view data about resource usage
  in pods and nodes.
    - `kubectl top pod --sort-by <JSONPATH> --selector <SELECTOR>`
    - Example:
        - `kubectl top pod --sort-by cpu`

- Can view resource usage by node
    - `kubectl top node`

- Checking to see if metrics server is running and responsive
    - `kubectl get --raw /apis/metrics.k8s.io/`






# Pods and Containers

## Application Configuration

Passing dynamic values to running applications/containers at runtime


###  ConfigMaps (cm)

- Docs: https://kubernetes.io/docs/concepts/configuration/configmap/
- Store *non-confidential* data in key-value pair format
- Not designed for large data (1MB max)
- Pods and ConfigMaps must be in the same namespace
- Multiple pods can reference the same ConfigMap
- Updates to ConfigMaps reflect in pod that consume it
    - Note: ConfigMaps consumed as env. vars are not updated automatically (require pod restart)
- ConfigMaps can be immutable, once created cannot be changed
- Example ConfigMap definition:
    - ```yaml
        apiVersion: v1
        kind: ConfigMap
        metadata:
          name: app-config
          namespace: default
        data:
          key1: value1               # <-- Each key can have value
          key2: value2
          key3:                      # <-- Can have tree structure
            subkey:
              more_keys: data
              even_more: more data
          key4: |
            You can have
            mulit-line
            data.
          key5.blah: |               # <-- File-like keys
            yo.cool=something
            yo.moo=something
        immutable: false
      ```
- Pods can consume ConfigMaps with the following:
    1. **Container Environmental variables**
        - Example:
            - ```yaml
                kind: Pod
                ...
                spec:
                  containers:
                    - ...
                      env:
                        - name: MY_ENV_VAR        # <-- Visible env. var in container
                          valueFrom:
                            configMapKeyRef:
                              name: my-configmap  # <-- Name of ConfigMap object
                              key: myKey          # <-- The value to read
                ...
               ```
    2. **Container command-line arguments**
    3. **Configuration files in a read-only volume**
        - Top level key will be the filename
        - Sub level keys will be placed inside the file
        - Example:
            - ```yaml
                kind: Pod
                ...
                spec:
                  volumes:
                    - name: config-vol
                      configMap:
                        name: my-configmap
                      ...
              ```

### Secrets

- Docs: https://kubernetes.io/docs/concepts/configuration/secret/
- Object that has small amount of *sensitive data* (password, token, key, etc)
- Protected from creating, viewing, or editing pods
- Not designed for large data (1MB max)
- NOTE: By default, secrets are stored un-encrypted in data store (etcd)
- When creating a secret using YAML file, the secret itself **must base64 encoded**
    - When secret is read by pod, it will be decoded
        - Example: `echo -n "some secret text" | base64`
    - When reading a secret, it has to be decoded back
        - Example: `echo "<ENCRYPTED TEXT>" | base64 -d`
- Pods can use secrets (same as ConfigMaps)
    1. **As files in a volume mounted to the container**
    2. **Container environmental variable**
    3. **Image pull authentication to authenticate to image registry**
- Data can also be stored as clear text using `stringData` instead of `data`
    - No `base64` encoding required
- Example Secret definition:
    - ```yaml
        apiVersion: v1
        kind: Secret
        metadata:
          name: my-secret
        type: Opaque                          # <-- Arbitary user-defined data
        data:
            username: 97sdf9==                # <-- base64 encoded text
            password: sd89sdfh/sd9f==         # <-- base64 encoded text
        immutable: false                      # <-- Default value is false
       ```
- Can load a secret from a file with declarative command
    - `kubectl create secret generic my-secret --from-file <LOCAL FILENAME>`



## Managing Container Resources

- **Resource Requests**
    - Define the amount of resources (CPU or memory) a container expected to have
    - Kubernetes scheduler will use this request to place pods in proper nodes
      with available resources
    - Containers are allowed to use more or less than the resource request
    - If request is much larger than any node can provide, pod can stay stuck in "Pending" status
    - NOTE: Memory is specified in Bytes, CPU is specified in CPU or millicpu units
        - 1 physical/virtual CPU core = 1 CPU
        - 0.5 CPU = 500m
    - Example:
        - ```yaml
            apiVersion: v1
            kind: Pod
            metadata:
              name: my-pod
            spec:
              containers:
                - name: busybox
                  image: busybox
                  resources:               # <-- Definition
                    requests:
                      cpu: "250m"          # <-- 0.25 CPU cores
                      memory: "128Mi"      # <-- 128Mi memory
          ```

- **Resource Limits**
    - Hard/Enforced resource limit for container
    - Container runtime will enforce this limit
    - Some container runtime terminate container processes attempting to use more than allowed resources
        - For example, Docker will throttle CPU to a value, while kill processes exceeding memory limit
    - Example:
        - ```yaml
            ...
            resources:
              limits:
                cpu: "250m"
                memory: "128Mi"
          ```


## Monitoring Container Health with Probes

Kubernetes can automatically detect unhealthy containers by actively monitoring container health.
`kubelet` uses different probes to gauge the health and readiness of the containers.

Docs: https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/


### Liveness Probe

- Allow to automatically determine whether or not a container application is healthy
- Monitor container on an ongoing bases over the lifetime of the container
- By default, will consider container down if the container process stops
- This mechanism can be customized
- Different type of liveness probes available:
    - `exec` - Run a command, if no error, success
    - `httpGet` - If status code is over 200 and below 400, success
    - `tcpSocket` - TCP check if port is open, if it is, success
- Example: `my-pod.yaml`
    - ```yaml
        apiVersion: v1
        kind: Pod
        ...
        spec:
          containers:
          - name: liveness
            ...
            livenessProbe:
              exec:                   # <-- exec type check
                command:              # <-- If command succeeds = healthy!
                - cat                 # <-- Command
                - /tmp/healthy        # <-- Arguments to command
              initialDelaySeconds: 5  # <-- Wait 5 seconds after container startup
              periodSeconds: 5        # <-- Run every 5 seconds
      ```


### Startup Probe

- Only run on container startup
- Stops once container is up and running.
- Can be useful on containers that have long startup times
- Example:
    - ```yaml
        apiVersion: v1
        kind: Pod
        ...
              startupProbe:
                initialDelaySeconds: 1
                periodSeconds: 2
                timeoutSeconds: 1
                successThreshold: 1
                failureThreshold: 30         # <-- Number of times allowed to fail
                httpGet:                     # <-- Check HTTP endpoint
                  scheme: HTTP
                  path: /
                  httpHeaders:
                    - name: Host
                      value: myapplication1.com
                  port: 80
      ```


### Readiness Probe

- Used to determine when container is ready to accept requests/traffic
- Traffic to a particular pod will not be send until all readiness checks have passed
- Readiness probe same as liveness and startup probe except: `readinessProbe:` block



## Self-Healing Pods with Restart Policies

Kubernetes allows you to customize when and how containers to be automatically restarted.
Three different values for restart policies: `Always`, `OnFailure`, and `Never`.

Can view pod restart status with `kubectl get pod <POD NAME>`

1. `Always` *(default)*
    - Containers always restarted if stopped
    - Even stopped when completed successfully
    - This is applications that always need to be running
    - Example:
        - ```yaml
            apiVersion: v1
            kind: Pod
            ...
            spec:
              restartPolicy: Always     # <-- Note
              container:
                ...
          ```

2. `OnFailure`
    - Container is unhealthy or exists with error code
    - Applications that need to run successfully and then stop

3. `Never`
    - No matter what, do not restart container
    - For containers that need to only run one time
    - If container fails, pod status is `Error`



## Multi-Container Pods

More than one container running inside a single pod. Containers share pod resources such as
network and storage. Containers can interact with each other.

- Keep containers in separate Pods unless they need to share resources
- Secondary container is sometimes called `sidecar`
- Shared resources
    - **Networking**
        - Same network namespace and can communicate on any port
        - Even if port is not exposed to the cluster
    - **Storage**
        - Using container volumes to share data in the pod
- Example: Two containers sharing one volume
    - ```yaml
        apiVersion: v1
        kind: Pod
        metadata:
            name: sidecar-pod
        spec:
            containers:
                - name: busybox1
                  image: busybox
                  command: ['sh', '-c', 'while true; do echo logs data > /output/output.log; sleep 5; done']
                  volumeMounts:
                    - name: sharedvol      # <-- Same shared volume
                      mountPath: /output#  # <-- Mounted here in container
                - name: sidecar
                  image: busybox
                  command: ['sh', '-c', 'tail -f /input/output.log']
                  volumeMounts:
                    - name: sharedvol      # <-- Same shared volume
                      mountPath: /input    # <-- Mounted here in container
            volumes:
              - name: sharedvol            # <-- The volume that is shared
                emptyDir: {}
      ```


## Init Containers

Docs: https://kubernetes.io/docs/concepts/workloads/pods/init-containers/

- Containers that run during startup process
- Listed in `InitContainer` spec section
- Run before any other containers in `containers` section
- Must run to once and to completion
- Run in the order they are listed
- When starting a pod, status will read `Init` if currently running init container
- Why?
    - Can be used to setup the pod, install tooling/utility
    - Can have container tasks and setup scripts not needed in the main containers
- Possible use cases
    - Wait for another Kubernetes resource to be created before startup
    - Perform sensitive startup steps securely outside of app containers
    - Populate data into a shared volume at startup. Main app container can read it.
    - Communicate with another service at startup
- Example: Init container with simple startup delay
    - ```yaml
        apiVersion: v1
        kind: Pod
        metadata:
          name: init-pod
        spec:
          containers:
            - name: nginx
              image: nginx:1.19.1
          initContainers:
            - name: delay
              image: busybox
              command: ['sleep', '30']
      ```

- Example: Wait for a service
    -```
        apiVersion: v1
        kind: Pod
        ...
        spec:
        ...
          initContainers:
            - name: my-init-container
              image: busybox:1.28
              command: ['sh', '-c', "until nslookup my-service; do echo waiting for my-service; sleep 2; done"]
     ```



# Scheduling in Kubernetes

Docs: https://kubernetes.io/docs/concepts/scheduling-eviction/kube-scheduler/

In Kubernetes, scheduling refers to making sure that Pods are matched to Nodes so that Kubelet can run them.

- **Scheduler**
    - Control plane component that handles scheduling
    - Watches newly created Pods and finds best Node for it
- Taken into account by scheduler:
    - Resource requests and available node resources
    - Various configurations that affect scheduling using node labels
        - ie. `nodeSelector`

## Pod Scheduling Configurations

### `nodeSelector`

- Limits which nodes the pod can be scheduled on
- Use labels to filter suitable nodes
- Can assign labels to nodes
    - `kubectl label nodes <NODE NAME> <KEY>=<VALUE>`
    - Example: `kubectl label nodes <worker1> mylabel=someValue`
- Example:
    - ```yaml
        apiVersion: v1
        kind: Pod
        ...
        spec:
          ...
          nodeSelector:
            mylabel: someValue      # <-- Only nodes with this label
      ```

### `nodeName`

- Bypass scheduling logic and assign pod to a specific Node by node name
- Example:
    - ```yaml
        ...
        spec:
          ...
          nodeName: worker0
      ```
- Getting node names
    - `kubectl get nodes`


## DaemonSets (ds)

DaemonSets will automatically runs a copy of a Pod on each node in the cluster

Docs: https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/

- Will run copy of the Pod on new nodes as they are added to the cluster
- Manages specified pods
- Will respect scheduling rules (ie. labels, taints, tolerations)
- Belong to `apiVersion: apps/v1`
- Actual pod description is defined in `template`
- Example: Single pod, single container in each node
    - ```yaml
        apiVersion: apps/v1         # <-- NOTE
        kind: DaemonSet
        metadata:
          name: my-daemonset
        spec:
          selector:
            matchLabels:
              app: my-daemonset     # <-- Any Pods that have this label
          template:                 # <-- The Pod template to create pods
            metadata:
              labels:
                app: my-daemonset   # <-- Typically matches the above selector
            spec:
              containers:
                - name: nginx
                  image: nginx:1.19.1
       ```
- Get list of DaemonSets
    - `kubectl get daemonset`
    - Will show how many desired/current/ready pods are running


## Static Pods

Static Pods are managed directly by kubelet daemon on a specific node, without the API server 
observing them.

Docs: https://kubernetes.io/docs/tasks/configure-pod-container/static-pod/

- Not managed by control plane, only kubelet watches each Static Pod.
- Can run even if there is no Kubernetes API server present
- Pod definition (YAML or JSON) is placed in a specific place on the node where kubelet will pick it up
    - This location is configurable
    - Default: `/etc/kubernetes/manifests/`
    - May need root permission to add files here
- Kubelet will automatically create "mirror Pod" which acts to make the Pod visible to 
  Kubernetes API server (But can't manage via API server)
    - Can see if with `kubectl get pods`





# Deployments

Defines a desired state (declaritive) for a ReplicaSet (set of replica Pods)

Docs: https://kubernetes.io/docs/concepts/workloads/controllers/deployment/

- Deployment controller seeks to maintain the desired state by creating, deleting, and replacing
  Pods with new configurations.
- Use cases
    - Scale application up or down by changing replica Pod number
    - Perform rolling updates to deploy new software version (ie. image versions)
    - Roll back to previous software version
- Includes the following configurations
    - `replicas` - Number of replica Pods that the Deployment will seek to maintin
    - `selector` - Label selector used to identify the replica Pods managed by the Deployment
    - `template` - Pod definition used to create all replica Pods for the Deployment
- Example:
    - ```yaml
        apiVersion: apps/v1      # <-- Note
        kind: Deployment
        metadata:
          name: nginx-deployment
          labels:
            app: nginx
        spec:
          replicas: 3            # <-- 3 Pods in this deployment
          selector:
            matchLabels:
              app: nginx         # <-- Get any pods with this metadata label
          template:              # <-- Definition of the Pod for this deployment
            metadata:            # <-- Note no "name:". Deployment will give name.
              labels:
                app: nginx       # <-- Match label in selector above
            spec:
              containers:
              - name: nginx
                image: nginx:1.14.2
                ports:
                - containerPort: 80
        ```
- Can change the image of a Deployment without YAML
    - `kubectl set image deployments/<DEPLOYMENT NAME> <IMAGE NAME AND VERSION>`
    - Example:
        - `kubectl set image deployments/my-deployment nginx:1.16.1`



## Scaling Applications with Deployments

Docs: https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#scaling-a-deployment

- Dedicating more or fewer resources to an application in order to meet changing needs.
- Useful in horizontal scaling
- Few ways of doing this
    - Change `replicas` value in YAML deployment description then `kubectl apply`
    - Use command `kubectl scale`
        - Example: `kubectl scale deployment/my-deployment --replicas=5`
    - Use command `kubectl autoscale` to scale depending on other factors
        - Example: Scaling based on CPU utilization
            - `kubectl autoscale deployment/my-deployment --min=10 --max=15 --cpu-percent=80`
- Can actively change a deployment
    - `kubectl edit deployment <DEPLOYMENT NAME>`
    - Change the deployment `replicas` in the `spec` section!



## Rolling Updates with Deployments

- **Rolling Updates**
    - Allows changes to Deployment at a controlled rate.
    - Gradually replacing old Pods with new Pods
    - Allows you to update your Pods without incurring downtime
    - This is by default triggered any time Deployment is updated

- **Rolback**
    - If update to deployment causes issues, you can roll back the deployment to previous
      working condition.

- Deployment rollout management commands
    - `kubectl rollout status deployment/<DEP. NAME>` - Checking deployment rolling update status
    - `kubectl rollout history deployment/<DEP. NAME>` - Checking deployment rollout history
    - `kubectl rollout undo deployment/<DEP. NAME>` - Undo previous deployment rollout
    - `kubectl rollout pause deployment/<DEP. NAME>` - Pause deployment rollout
    - `kubectl rollout resume deployment/<DEP. NAME>` - Resume paused deployment rollout




# Networking

Docs: https://kubernetes.io/docs/concepts/services-networking/


## Network Model

Docs: https://www.ibm.com/docs/en/cloud-private/3.1.2?topic=networking-kubernetes-network-model

- Set of standards that define how networking between Pods behaves
- Variety of this model implementation
    - Calico network plugin
- Define how pods communicate with each other
- Each pod has its own unique IP address within the cluster, even in a different node!
    - A pod can reach any other pod using pod's IP
    - Pod IPs drawn from IP pool created at installation time
- Kubernetes services can expose application on Pod to be reachable outside of the cluster


## Container Network Interface (CNI) Plugins

Docs: https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/

- Type of Kubernetes network plugins
- Provide network connectivity between Pods
- Adhere to the standard set by the Kubernetes network model
- Plugins will depend on specific situation
- Each plugin installation process may differ
- Nodes will remain in `NotReady` state until a network plugin is installed
- Example network plugin installation
    - `kubectl apply -f <LOCAL FILE OR REMOTE URL YAML>`
    - `kubectl apply -f https://docs.projectcalico.org/manifests/calico-typha.yaml`


## Domain Name System (DNS) in Kubernetes

Docs: https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/

- Allows Pods to locate other Pods and Services using domain names (ie. `some-name` instead of IP address (ie. `192.168.1.3`)
- DNS runs as a service within the cluster
- Typically in `kube-system` namespace
- kubeadm clusters
    - Use CoreDNS as DNS solution
        - Typically can see them
            - Pods: `kubectl get pods -n kube-system`
            - Service: `kuabectl get service -n kube-system`
    - Automatically given domain name of the following form
        - `<POD IP>.<NAMESPACE NAME>.pod.cluster.local`
        - NOTE: Pod IP address `.` replaced by `-`
            - Example: `192-168-10-100.default.pod.cluster.local`
        - Another pod can communicate with this pod using this DNS name from any namespace
    - More useful with kubernetes services


## NetworkPolicies (netpol)

Kubernetes object that allows you to control the flow of network communication to and from the Pods

Docs: https://kubernetes.io/docs/concepts/services-networking/network-policies/

- Allows for a more secure network
- Can isolate the Pod from traffic that is not needed
- **By default, pods are considered non-isolated and completely open to all traffic**
- NetworkPolicy can apply to Ingress (incoming), Egress (outgoing), or both types of network traffic
    - Ingress traffic => `from`
    - Egress traffic => `to`
    - Can use different selectors for `from` and `to`
        1. `podSelector` - Traffic from and to specific pods
        2. `namespaceSelector` - Traffic from and to specific namespaces
        3. `ipBlock` - Traffic from a specific IP range using CIDR notation (ie. 10.0.1.0/16)
    - `port` - Specify one or more ports that will allow traffic, includes protocol (ie. `TCP`)
- Example:
    - ```yaml
        apiVersion: networking.k8s.io/v1   # <-- Note
        kind: NetworkPolicy
        metadata:
          name: test-network-policy
          namespace: default
        spec:
          podSelector:                   # <-- Which pod to apply this to, same namespace
            matchLabels:
              role: db
          policyTypes:                   # <-- Should batch the sections below
          - Ingress                      # <-- If "ingress" section omitted, no in-traffic
          - Egress                       # <-- If "egress" section omitted, no out-traffic
          ingress:
          - from:                        # <-- "from" for ingress only
            - ipBlock:
                cidr: 172.17.0.0/16      # <-- Allow this IP range
                except:
                - 172.17.1.0/24
            - namespaceSelector:
                matchLabels:
                  project: myproject     # <-- Allow from name space with this label
            - podSelector:
                matchLabels:
                  role: frontend         # <-- Allow Pods with this label
            ports:                       # <-- Rules applied only to these ports/protocol
            - protocol: TCP
              port: 6379
          egress:
          - to:                          # <-- "to" for egress only
            - ipBlock:
                cidr: 10.0.0.0/24
            ports:
            - protocol: TCP
              port: 5978
        ```



## Troubleshooting Network

- Network is not set up
    - **ISSEUS**:
        - Cluster Notes status is `NotRead` network is not set up
        - Pod `IP` is `<none>`and/or `STATUS` is `Pending`, network is not set up
        - `kubectl describe node <NODE NAME>` shows event of `Starting kube-proxy`
        - `kubectl get pods -n kube-system` shows no network plugin pod (ie. `calico*`)
    - **SOLUTIONS**
        - `kubectl apply -f <NETOWRK PLUGIN YAML>`

- Misconfigured NetworkPolicies
    - **ISSUES**
        - Within the same namespace, pods cannot communicate with each other
        - `curl <IP ADDRESS>` from one pod cannot reach another pod
    - **SOLUTIONS**
        - Check network policties
            - `kubectl get networkpolicy`
            - `kubectl describe networkpolicy <NAME>`
        - Adjust all NetworkPolicies
            - `kubectl edit networkpolicy -n <NAMESPACE> <NAME>`
            - `kubectl apply -f <NETWORK POLOCY YAML>`

- All open traffic between Pods, namespaces, IP
    - **ISSUES**
        - Pods ARE able to communicate with a specific Pod, Namespace, IP ranges
    - **SOLUTIONS**
        - No NetworkPolicies are set up, set them up





# Services

Services expose applications running as a set of Pods.

Docs: https://kubernetes.io/docs/concepts/services-networking/service/

- Clients are not aware of Pods (creates abstraction)
- Client traffic to a Kubernetes service is routed to its Pods in a load-balanced fashion
    - **Traffic**: *Client -> Service -> Endpoint -> Pods in cluster*
- **Endpoints**
    - Backend entities to which services route traffic
    - Services that route to multiple Pods, each Pod has an endpoint for that Service
    - Look at Service's Endpoints to determine Service-Pod traffic routing
        - `kubectl get endpoints <SERVICE NAME>`


## Types of Services

How and where the Service will expose the application.


### ClusterIP (default)

- Expose applications *inside* the cluster network
- Clients will be other Pods within the cluster
- **Traffic:** *Pod --> Service --> Endpoint --> Pods*
- Can create a starting template:
    - `kubectl create service clusterip svc-internal --tcp=80:80 --dry-run='client' -o yaml > svc-internal.yml`
    - Remember, you can set up command completion, and use `--help`
    - `--tcp=<INSIDE PORT>:<OUTSIDE TARGETPORT`
- Example: Service exposed within a cluster
    - ```yaml
        apiVersion: v1            # <-- Note
        kind: Service
        metadata:
          name: svc-clusterip
        spec:
          type: ClusterIP         # <-- Can leave out, it is default anyways
          selector:
            app: svc-example      # <-- Locate and attach to Pods with this label
          ports:                  # <-- Can have many ports
            - protocol: TCP
              port: 80            # <-- Service listens on this port (outside pod)
              targetPort: 80      # <-- Port on Pods attached to this service (inside pod)
      ```


### NodePort

- Expose application *outside* the cluster network
- Applications or users are accessing application from outside the cluster
- Can be accessed using the Node's host IP address (ie. `<NODE IP>:<NodePort>`
- Kubernetes allocates a port from a range of (default: 30000-32767) to service
    - *Same port on every Node*
    - For example, port 30020, on all Nodes running the Service
    - Can specify with `nodePort` key
- Can create a starting template:
    - `kubectl create service nodeport svc-external --tcp=80:80 --dry-run=client -o yaml > svc-external.yml`
    - Remember, you can set up command completion, and use `--help`
- Example: Service exposed outside a cluster
    - ```yaml
        apiVersion: v1
        kind: Service
        metadata:
          name: my-service
        spec:
          type: NodePort        # <-- NOTE
          selector:
            app: MyApp
          ports:
              # By default and for convenience, the `targetPort` is set to the same value as the `port` field.
            - port: 80          # <-- Service listens on this port (outside pod)
              targetPort: 80    # <-- Ports on Pods attached to this service (inside pod)
              nodePort: 30007   # <-- Exposed port on nodes. Optional, by default chosen 30000-32767 (outside cluster)
      ```


### LoadBalancer
    - Expose applications *outside* of cluster network
    - Use external cloud load balancer
    - Only works with cloud platforms (ie. AWS) that include load balancing
    - **Traffic:**: *Client -> LoadBalancer -> Cluster/Service -> Endpoint -> Pod*

### ExternalName
    - No proxying of any kind is set up
    - Maps Service to contents of `externalName` field (ie. foo.bar.example.com)
    - **Not covered in CKA exam**




## Service DNS Names

Docs: https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/

- Kubernetes assigns DNS names to Services, allowing applications within the cluster to easily
locate them
- DNS query in different namespaces may return different results
- Fully Qualified Domain Name (FQDN) has the following format:
    - `<SERVICE NAME>.<NAMESPACE NAME>.svc.<CLUSTER DOMAIN>`
    - Default `<CLUSTER DOMAIN>` is `cluster.local`
    - Examples:
        - `my_service.default.svc.cluster.local`
        - `my_service.testing.svc.my_company.com`
- FQDN can be reached form any Namespace in the entire cluster
- **NOTE**: Pods within the **same** Namespace can simply use the service name
    - Example: `curl my-service`



## Ingress (ing)

Docs: https://kubernetes.io/docs/concepts/services-networking/ingress/

- Ingress (incoming) is a Kubernetes object that manages external access to Services in the cluster
- Typically HTTP of HTTPS routes from the outside of the cluster
- *More functionality than a simple `NodePort` Service*
- Can manage SSL termination, advanced load balancing, or name-based virtual hosting
- **Traffic:** *Client -> Ingress -> Service -> Endpoint -> Pods*
- Can create starting template:
    `kubectl create ingress <NAME> --path:<SERVICE>:<PORT> [OPTIONS]`
- Define a set of **routing rules**
    - Routing rule properties determine to which requests it applies to
    - Each routing rule has set of **paths** that corresponds to a backend service
    - Requests that matches the path will be routed to the backend
    - Example:
        - ```yaml
            apiVersion: networking.k8s.io/v1
            kind: Ingress
            metadata:
              name: my-ingress
            spec:
              rules:
                - http:
                    paths:
                      - path: /somepath       # <-- When this path is requested
                        pathType: Prefix
                        backend:
                          service:
                            name: my-service  # <-- Route to this service, can be ClusterIP-based
                            port:
                              number: 80      # <-- To this service port
          ```
        - Request: `http://some-endpoint.com/somepath`
        - Will be routed to Service `my-service` at port `80`
- If service uses **named ports**, Ingress can use the port name for `port`
    - Example:
        - ```yaml
            ...
                  backend:
                    service:
                      name: my-service
                      port:
                      name: http-port   # <-- In Pod: spec.ports.name
          ```
- Check Ingress: `kubectl describe ingress <INGRESS NAME>`
- `pathType`
    - Docs: https://kubernetes.io/docs/concepts/services-networking/ingress/#path-types
    - `Exact` - Matches URL path exactly and is case sensitive
    - `Prefix` - Mataches based on URL path prefix split by `/`. (ie. `/yo` matches `/yo/blah`)


### Ingress Controllers

Docs: https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/

- Ingress objects can't do anything themselves
- Must have one or more *Ingress Controllers*
- Used with *Ingress Class*
    - https://kubernetes.io/docs/concepts/services-networking/ingress/#ingress-class
- Variety of Ingress Controllers, with different implementation methods for external access to Service
- Commonly and easily installed with Helm Charts
- Can add annotations to Ingress object referencing Ingress Controller functionality
- Most have UI with them available
- **Example:** NGINX Ingress Controller (https://kubernetes.github.io/ingress-nginx/)
    - Does not require third-party modules to run
    - Simplest to set up and use
    - Best for beginners
    - Does not support dynamic design, reload needed after endpoint change
- **Example:** Traefik (https://traefik.io/)
    - Support for TCP, HTTP, HTTPS, and GRPC
    - Supports round-robin and weighted round-robin for load balancing
    - Supports *Let's Encrypt*
    - Setup requires setting up ServiceAccount, ClusterRole, Deployment for Traefik
- **Example:** Istio (https://istio.io/)



# Storage

- Container file system is ephemeral/temporary and is deleted when container is removed or 
  recreated
- To hold on to data persistantly, we need a non-ephemeral solution


## Volumes

Allow storage of data outside of the container file system while allowing the container to
access the data at runtime.

Docs: https://kubernetes.io/docs/concepts/storage/volumes/

- Volume belongs to the Pod and there for the life of the Pod
- Available even if container is removed, data is still there
- Simple container external storage
- Can be set up with Pod/container specification
- Mounting the same volume into multiple containers allows for sharing of data between containers
  on the same Pod
    - Could have a "sidecar" container with special tools to process data from main container
- **Example**: Pod directory mounted into the container
    - ```yaml
        apiVersion: v1
        kind: Pod
        metadata:
          name: pod-with-volume
        spec:
          containers:
            - name: busybox
              image: busybox
              volumeMounts:            # <-- List of mounts within container
                - name: my-volume      # <-- Reference volume in volume section
                  mountPath: /output   # <-- Directory within the container
            - name: busybox2           # <-- Can have more than one container
              image: busybox
              volumeMounts:
                - name: my-volume      # <-- Same volume to share data between containers
                  mountPath: /input
          volumes:
            - name: my-volume
              hostPath:                # <-- Reference dir/file on host Node filesystem
                path: /var/data        # <-- Directory on Pod
            - name: my-empty-dir
              emptyDir: {}             # <-- Temp. empty directory on Node only for life of Pod
      ```


## Volume Types

Docs: https://kubernetes.io/docs/concepts/storage/volumes/#volume-types

Volumes and Persistent Volumes have a **volume type**, which determines how the storage is
actually handled (mechanism for storage).

Various volumes types support storage methods such as:

- Network File System (NFS)
- Cloud storage (AWS, Azure, GCP, etc)
- Kubernetes ConfigMaps and Secrets
- Simple directory on the cluster Nodes


### Common Volume Types

- `hostPath`
    - Stores data in a specified file or directory on the host Node
    - `type` can be:
        - `Directory`
        - `File`
        - `Socket`
        - more
    - `path` is the directory on the Node
- `emptyDir: {}`
    - Directory that exists only as long as the Pod exists on the Node
    - Directory and its data are deleted when Pod is removed
    - Useful for simply sharing data between containers in same Pod
- `configMap`
    - Inject configuration data into pods
    - Provide ConfigMaps name in the volume section
    - **Example**:
        - ```yaml
          volumes:
            - name: config-vol
              configMap:
                name: log-config     # <-- Name of the ConfigMap
                items:
                  - key: log_level
                    path: log_level
          ```




## Persistent Volumes

Allows treating storage as an abstract resource


### PersistantVolume (pv)

Docs: https://kubernetes.io/docs/concepts/storage/persistent-volumes/

- Treats storage as abstract resource for Pods
- Describes the underlying storage resource (local, cloud, etc)
- Tell Kubernetes what you need and it will allocate as you need it
- Storage in the cluster that has been provisioned by an admin or dynamically provisioned
  using Storage Classes
- `storageClassName` references a `StorageClass` object
- `persistentVolumeReclaimPolicy` determines how the storage resources can be reused when the
  PersistentVolume's associated PersistentVolumeClaims are deleted
    - `Retain` - Keep all data. Manual clean up and prepare for reuse
    - `Recycle` - Delete all data. Basic scrub (`rm -rf /my-volume/*`)
    - `Delete` - Associated storage asset is deleted (only for cloud resources)
- Check status of PersistentVolume
    - `kubectl get pv`
    - Statuses
        - `Available` - Not bound to PersistentVolumeClaim
        - `Bound` - Bound to a PersistentVolumeClaim
        - `Released` - Claim has been deleted, resources not yet reclaimed by cluster
        - `Failed` - Voulme failed automatic reclamation
- **Example:**
    - ```yaml
        apiVersion: v1
        kind: PersistentVolume
        metadata:
          name: my-pv
        spec:
          storageClassName: localdisk             # <-- References a StorageClass
          persistentVolumeReclaimPolicy: Recycle  # <-- Scrub and reuse
          capacity:
            storage: 1Gi                          # <-- Total available to be able to claim
          accessModes:
            - ReadWriteOnce                       # <-- Must match PersistentVolumeClaim!
          hostPath:                               # <-- Type of storage
            path: /var/output
      ```


### StorageClass (sc)

Docs: https://kubernetes.io/docs/concepts/storage/storage-classes/

- Allows admins to specify types of storage services offered on their platform
- Different `parameters` may be accepted depending on the `provisioner`
    - Example: For provisioner `kubernetes.io/aws-ebs` we can have the following
        - ```yaml
            provisioner: kubernetes.io/aws-ebs
            parameters:
              type: io1
              iopsPerGB: "10"
              fsType: ext4
          ```
- Use case could include creating a low-performance and high-performance storage type
- Kubernetes users can then choose StorageClass that fit need of application
- `allowVolumeExpansion` determines whether or not the StorageClass supports the ability to resize
  volumes after they are created
    - If set to `false` and try to resize, will get an error
    - By default, set to `false`
- If not StorageClass is created, one will be automatically be created
- **Example:** Storage Class for local storage
    - ```yaml
        apiVersion: storage.k8s.io/v1              # <-- Note
        kind: StorageClass
        metadata:
          name: localdisk
        provisioner: kubernetes.io/no-provisioner  # <-- Type of service/platorm/provider
        allowVolumeExpansion: true                 # <-- Volume can be resized after creation
      ```


### PersistentVolumeClaim (pvc)

- Request for storage by a user.
- PersistentVolumeClaims consume and bind to PersistentVolume resources
- References Order:
    - **Pod -> PersistentVolumeClaim -> PersistentVolume -> StorageClass -> External Storage**
    - Define/Create in reverse!
- Claims can request specific storage size and access modes
- PersistentVolume and PersistentVolumeClaims are bound
- Will look for PersistentVolume that is able to meet requested criteria
    - If found, it will bind to it
- **Example:**
    - ```yaml
        apiVersion: v1
        kind: PersistentVolumeClaim
        metadata:
          name: my-pvc
        spec:
          storageClassName: localdisk     # <-- Reference StorageClass
          volumeMode: Filesystem          # <-- Default value
          accessModes:
            - ReadWriteOnce               # <-- Must match PersistentVolume!
          resources:
            requests:
              storage: 100Mi              # <-- Storage claimed from PersistentVolume
      ```
- Finally, the PersistentVolumeClaim is mounted in as a volume for a Pod
    - **Example:**
        - ```yaml
            apiVersion: v1
            kind: Pod
            metadata:
              name: pv-pod
            spec:
              containers:
                - name: busybox
                  image: busyboxs
                  volumeMounts:
                    - name: pv-storage
                      mountPath: /output    # <-- Volume mounted here
              volumes:
                - name: pv-storage
                  persistantVolumeClaim:
                    claimName: my-pvc       # <-- Reference to pvc
          ```
- Can edit PersistentVolumeClaim to expand storage
    - `kubectl edit pvc <PVC NAME>`
    - `kubectl apply -f <EDITED YAML FILE>`
    - Change `resources.requests.storage`









# Troubleshooting


## Kubernetes API server down

- Symptoms:
    - `kubectl` cannot interact with cluster
    - Error Message
        - `The connection to the server <ADDRESS>:<PORT> was refused = did you specify the right host or port?`
- Possible fixes:
    - Makes sure docker and kubelet services are up and running on control node
    - `docker --version`
    - `sudo systemctl status kubelet`


## Node is having problems

1. Check Node status
    - `kubectl get nodes`
    - `kubectl describe node <NODE NAME>`

2. Check status and/or starting/enabling services
    - For `docker` and `kubelet`
    - Status: `sudo systemctl status kubelet`
    - Start: `sudo systemctl start kubelet`
    - Enable: `sudo systemctl enable kubelet`


## Cluster System Pod is having problems

`kubeadm` cluster, several kubernetes components run as pods in `kube-system` namespace

1. Check `kube-system` component status
    - `kubectl get pods -n kube-system`
    - `kubectl describe pod <POD NAME> -n kube-system`


## Kubernetes Service Logs

Can check logs for Kubernetes related services on each node using `journalctl`

- `sudo journalctl -u kubelet`
- `sudo journalctl -u docker`

`SHIFT-g` to jump to the end of logs


## Cluster Component Logs

Kubernetes cluster components have log output redirected to `/var/log`

- `/var/log/kube-apiserver.log`
- `/var/log/kube-scheduler.log`
- `/var/log/kube-controller-manager.log`

Note, `kubeadm` clusters may not have these components because components run inside container.
In that case, access with `kubectl logs <COMPONENT POD> -n kube-system`


## Application Issues

1. Getting status
    - `kubectl get pod`
    - `kubectl describe pod <POD NAME>`

2. Run command inside container in pod
    - `kubectl exec <POD NAME> -c <CONTAINER NAME> -- <COMMAND>`

3. Interactive shell inside the pod container
    - `kubectl exec <POD NAME> -c <CONTAINER NAME> -i -t -- sh`

4. Get container logs
    - `kubectl logs <POD NAME> -c <CONTAINER NAME>`


## Networking Issues

1. Check if kubernetes networking plugin is up and running
    - `kubectl get pods --all-namespaces`

2. Check `kube-proxy`
    -`kube-proxy` runs inside `kube-system` namespace
    - Can check logs for it like any other pod `kubectl logs`
        - `kubectl logs kube-porxy-XXXXX`

3. Check kubernetes DNS
    - Runs inside `kube-system` namespace
    - `kubectl logs coredns-XXXXXX-XXXX`

### `netshoot` Tool

- Separate container that can test and gather information about network functionality
- Image: `nicolaka/netshoot`
- Variety of networking exploration and troubleshooting tools
- More info: https://github.com/nicolaka/netshoot
- **Steps**
    1. Create the `netshoot` container
        - Example: Basic `netshoot` pod
            - ```yaml
                apiVersion: v1
                kind: Pod
                metadata:
                  name: nginx-netshoot
                spec:
                  containers:
                    - name: nginx
                      image: nicolaka/netshoot
                      command: ['sh', '-c', 'watch ls']
              ```

    2. Create `netshoot` container and use `kubectl exec` into `netshoot` container to explore network
        - `kubectl exec -i -t netshoot -- sh`

    3. Explore network
        - `curl <NETWORK RESOURCE>` - HTTP/HTTPS request
        - `ping <NETWORK RESOURECE>` - Check is something is up and reachable
        - `nslookup <NETWORK RESOURECE>` - Get DNS info on a IP or FAQN URL
        - `netstat`
        - `python3`
        - ... much more





# Tips and Tricks

- Create a sample YAML file of the resource to modify later using `--dry-run -o yaml`
    - Example: Console out the YAML file for a deployment
        - `kubectl create deployment my-deployment --image=nginx --dry-run=client -o yaml`

- Get the definition of a currently run pod
    - `kubectl get pod <POD NAME> -o yaml > my-pod.yml`

- Record a command using `--record` to add to the object's describe description.
    - **NOTE**: *This feature will be removed in future Kubernetes version*
    - Allows for later review of object
    - This will appear when using `describe` under `Annotations:`
    - Example: `kubectl scale development my-development replicas=5 --record`

- Use sample/template resource YAML configuration as a base, then add to it
    - Sample/templates can be found in the official Kubernetes docs.

- Can edit an active resource object using `kubectl edit <RESOURCE TYPE> <RESOURCE NAME>`
    - Will open a text editor (ie. VIM)
    - Example: `kubectl edit deployment my-deployment`
    - Example: `kubectl edit networkpolicy -n some-namespace my-networkpolicy`

- Adding CLI command completion to shell
    - `kubeadm completion bash >> .bashrc`
    - `kubectl completion bash >> .bashrc`
    - Reload current shell (`exec bash`) or open a new shell

- Check reaching a pod/service from temporary pod
    - Need pod with curl installed
    - Describe simple pod with `nikolaka/netshoot` image
    - ```yaml
        apiVersion: v1
        kind: Pod
        metadata:
          name: netshoot
        spec:
          containers:
            - name: netshoot
              image: nicolaka/netshoot
              command: ['watch', 'ls']     # <-- Runs "ls" every 2 seconds
      ```
    - if `curl` is not there, run `wget` form this pod to other objects
        - To other pod: `kubectl exec curl-pod -- wget -O - <POD IP>:<PORT OF POD>`
        - To service: `kubectl exec curl-pod -- wget -O - <SERVICE NAME>:<ClusterIP PORT>`

- Check DNS entries within a Pod
    - `kubectl exec <POD NAME> -- nslookup <IP ADDRESS>`
    - Example:
        - `kubectl exec my-test-pod -- nslookup 10.104.162.248`
        - Response shows the DNS entry for that IP address

- Get specific field form YAML output using `yq`
    - `<COMMAND> -o yaml | yq r - <TOP LEVEL KEY>.<SUB KEY>.<SO ON>`
    - Example:
        - `kubectl get secret credentials -n demo -o yaml | yq r - data.password`
        - Note, in this case for secret, need `| base64 -d`






# Random

- Setting Custom Hostname
    - `sudo hostnamectl set-hostname k8s-control`

- Set up host file
    - `sudo vim /etc/hosts`
    - Add private IPs for all servers
    - `<PRIVATE IP> <HOSTNAME>`

