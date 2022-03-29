<p align="center"><img width="150" alt="portfolio_view" src="logo.png"></p>
<!-- <h4 align="center"><a href="HYPERLINK HERE">URL HERE</a></h3> -->
<h1 align="center">Certified Kubernetes Administrator Notes</h1>




# Exam

- Kubernetes 1.23
- Ubuntu 20.04
- Terminal
    - VIM v???
    - tmux?



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
    - ``
      cat << EOF | sudo tee /etc/modules-load.d/containerd.conf
      overlay
      br-netfilter
      EOF
      ``
- Enable the kernel modules right now
    - `sudo modprobe overlay`
    - `sudo modprobe br-netfilter`
- System level networking settings
    - ``
        cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
            net.bridge.bridge-nf-call-iptables = 1
            net.ipv4.ip_forward = 1
            net.bridge.bridge-nf-call-ip6tables = 1
        EOF
      ``

- Command to make networking settings take effect immediately
   - `sudo sysctl --system`

- Install **containerd**
    - `sudo apt-get update && sudo apt-get install -y containerd`

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
    - `sudo apt-get update && sudo apt-get install -y apt-transport-https curl`

- Add GPG key for the Kubernetes repository
    - `curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -`

- Setup the Kubernetes repository entry in `apt`
    - ``
        cat << EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
        deb https://apt.kubernetes.io/ kubernetes-xenial main
        EOF
      ``

- Fetch newly added repository information
    - `sudo apt-get update`

- Install Kubernetes tools `(note the version)`
    - List of kubernetes versions: https://kubernetes.io/releases/
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



# Namespaces

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

- Can specify to look at all namespaces with `--all-namespaces`
    - Can be slow to complete
    - Example `kubectl get pods --all-namespaces`

- Create a custom namespace
    - `kubectl create namespace my-namespace`

- Set a default namespace for all subsequent commands
    - `kubectl config set-context --current --namespace=<insert-namespace-name-here>`




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
    - `sudo apt-get update && sudo apt-get install -y --allow-change-held-packages kubeadm=1.22.2-00`

3. Plan the cluster upgrade
    - `sudo kubeadm upgrade plan v1.22.2`

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
    - *This command is run on the control plane node*
    - `kubectl drain <WORKER NODE NAME> --ignore-daemonsets`
    - Same as the control plane node command for draining
    - May have to use `--force`

2. Upgrade `kubeadm` CLI tool
    - `sudo apt-get update && sudo apt-get install -y --allow-change-held-packages kubeadm=1.22.2-00`

3. Upgrade the `kubelet` configuration
    - `sudo kubeadm upgrade node`

4. Upgrade `kubelet` and `kubectl` CLI tools

5. Uncordon the node, putting it back into cluster service
    - *This command is run on the control plane node*
    - `kubectl uncordon <WORKER PLANE NODE NAME`



## Backing up and Restoring `etcd` Cluster Data

`etcd` is the backend key-value data storage solution for Kubernetes cluster.
All Kubernetes objects, applications, and configurations are stored in etcd.

- etcd website: https://etcd.io/

- Need to use `etcdctl` CLI tool
    - Select API version with `ETCDCTL_API` environmental variable
    - `etcdctl get <KEY>` - Get a specific value for a key

- etcd typically runs on port 2379

- If TLS encryption, must specify certificates
    - Example: Looking up value for `cluster.name` key
        - ``
            ETCDCTL_API=3 etcdctl get cluster.name \
              --endpoints=https://10.0.1.101:2379 \
              --cacert=/home/cloud_user/etcd-certs/etcd-ca.pem \
              --cert=/home/cloud_user/etcd-certs/etcd-server.crt \
              --key=/home/cloud_user/etcd-certs/etcd-server.key
          ``

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

- **Command: `kubectl api-resources`**
    - List all available Kubernetes resource objects for current Kubernetes version.

- **Command: `kubectl get`**
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

- **Command: `kubectl describe`**
    - Show detailed and human readable information of a specific resource
    - Docs: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#describe
    - Examples:
        - `kubectl describe pods/nginx`
        - `kubectl describe -f pod.json` - Pod defined by `pod.json`

- **Command: `kubectl create`**
    - Create a Kubernetes resource from file or stdin
    - Docs: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#create
    - If object exists, error occurs
    - Examples:
        - `kubectl create -f pod.yaml`
        - `cat pod.json | kubectl create --filename -`

- **Command: `kubectl apply`**
    - Will create a new object or modify an existing one
    - Docs: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
    - Will not throw error if object exists
    - Examples:
        - `kubectl apply -f pod.yaml`
        - `kubectl apply -k my-dir/` - Will look into `my-dir` directory for all resource configs

- **Command: `kubectl delete`**
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

- **Command: `kubectl exec`**
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


## `kubectl` Tricks

- Create a sample YAML file of the resource to modify later using `--dry-run -o yaml`
    - Example: Console out the YAML file for a deployment
        - `kubectl create deployment my-deployment --image=nginx --dry-run=client -o yaml`

- Get the definition of a currenlty run pod
    - `kubectl get pod <POD NAME> -o yaml > my-pod.yml`

- Record a command using `--record` to add to the object's describe description.
    - Allows for later review of object
    - This will appear when using `describe` under `Annotations:`
    - Example: `kubectl scale development my-development replicas=5 --record`
    - **NOTE**: *This feature will be removed in future Kubernetes version*

- Use sample/template resource YAML configuration as a base, then add to it
    - Sample/templates can be found in the official Kubernetes docs.


## Role-Based Access Control (RBAC) Authorization

Role-based access control (RBAC) is a method of regulating access to resources based on
the roles of individual users within the organization. Essentially, what users are allowed
to do and access within the cluster.

Docs: https://kubernetes.io/docs/reference/access-authn-authz/rbac/


### RBAC Objects

1. **Role**
    - Sets permissions within a particular namespace
    - Can specify `namespace`
    - Example: Role definition YAML file
        - ``
            apiVersion: rbac.authorization.k8s.io/v1
            kind: Role
            metadata:
              namespace: default    # <-- Note namespace defined
              name: pod-reader
            rules:
            - apiGroups: [""]       # <-- "" indicates the core API group
              resources: ["pods", "pods/logs"]
              verbs: ["get", "watch", "list"]
          ``

2. **ClusterRole**
    - Not namespace specific, cluster-wide
    - Do not need to specify `namespace`

3. **RoleBinding**
    - Grants permission defined in a role to a user or set of users
    - Holds a list of subjects (users, groups, or service accounts)
    - Holds reference to the role being granted
    - Can only reference an Role in the same namespace
    - Can also reference a ClusterRole
    - Example:
        - ``
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
            ``

4. **ClusterRoleBinding**
    - To bind a ClusterRole to all namespaces, use ClusterRoleBinding


### User Permissions


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



## Service Accounts

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
    - TODO




## Inspecting Resource Usage

Kubernetes Metrics Server
- Need add-on to collect and provide metrics data about resources, pods, and containers
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

Ways of doing this:

1. **ConfigMaps**
    - Docs: https://kubernetes.io/docs/concepts/configuration/configmap/
    - Store *non-confidential* data in key-value pair format
    - Not designed for large data (1MB max)
    - Pods and ConfigMaps must be in the same namespace
    - Multiple pods can reference the same ConfigMap
    - Updates to ConfigMaps reflect in pod that consume it
        - Note: ConfigMaps consumed as env. vars are not updated automatically (require pod restart)
    - ConfigMaps can be immutable, once created cannot be changed
    - Pods can consume ConfigMaps with the following:
        1. *Container Environmental variables*
            - Example:
                - ``
                    kind: Pod
                    ...
                    spec:
                      containers:
                        - ...
                          env:
                            - name: MY_ENV_VAR
                              valueFrom:
                                configMapKeyRef:
                                  name: my-configmap
                                  key: myKey
                    ...
                   ``
        2. *Container command-line arguments*
        3. *Configuration files in a read-only volume*
            - Top level key will be the filename
            - Sub level keys will be placed inside the file
            - Example:
                - ``
                    kind: Pod
                    ...
                    spec:
                      volumes:
                        - name: config-vol
                          configMap:
                            name: my-configmap
                          ...
                  ``
    - Example ConfigMap definition:
        - ``
            apiVersion: v1
            kind: ConfigMap
            metadata:
              name: app-config
              namespace: default
            data:
              key1: value1
              key2: value2
              key3:
                subkey:
                  more_keys: data
                  even_more: more data
              key4: |
                You can have
                mulit-line
                data.
            immutable: false
          ``

2. **Secrets**
    - Docs: https://kubernetes.io/docs/concepts/configuration/secret/
    - Object that has small amount of *sensitive data* (password, token, key, etc)
    - Protected from creating, viewing, or editing pods
    - Not designed for large data (1MB max)
    - NOTE: By default, secrets are stored un-encrypted in data store (etcd)
    - When creating a secret using YAML file, the secret itself **must base64 encoded**
        - When secret is read by pod, it will be decoded
        - Example: `echo -n "some secret text" | base64`
    - Pods can use secrets
        1. **As files in a volume mounted to the container**
        2. **Container environmental variable**
        3. **Image pull authentication to authenticate to image registry**
    - Example Secret definition:
        - ``
            apiVersion: v1
            kind: Secret
            metadata:
              name: my-secret
            type: Opaque
            data:
                username: user
                password: sd89sdfh/sd9f   # <-- base64 encoded text
           ``
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
        - ``
            apiVersion: v1
            kind: Pod
            metadata:
              name: my-pod
            spec:
              containers:
                - name: busybox
                  image: busybox
                  resources:           <-- Definition
                    requests:
                      cpu: "250m"      <-- 0.25 CPU cores
                      memory: "128Mi"
          ``

- **Resource Limits**
    - Hard/Enforced resource limit for container
    - Container runtime will enforce this limit
    - Some container runtime terminate container processes attempting to use more than allowed resources
        - For example, Docker will throttle CPU to a value, while kill processes exceeding memory limit
    - Example:
        - ``
            ...
            resources:
              limits:
                cpu: "250m"
                memory: "128Mi"
          ``


## Monitoring Container Health with Probes

Kubernetes can automatically detect unhealthy containers by actively monitoring container health.
`kubelet` uses different probes to gauge the health and readiness of the containers.

Docs: https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/

- **Liveness Probe**
    - Allow to automatically determine whether or not a container application is healthy
    - Monitor container on an ongoing bases over the lifetime of the container
    - By default, will consider container down if the container process stops
    - This mechanism can be customized
    - Different type of liveness probes available:
        - `exec` - Run a command, if no error, success
        - `httpGet` - If status code is over 200 and below 400, success
        - `tcpSocket` - TCP check if port is open, if it is, success
    - Example: `my-pod.yaml`
        - ``
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
          ``

- **Startup Probe**
    - Only run on container startup and stop once container is up and running.
    - Can be useful on containers that have long startup times
    - Example:
        - ``
            apiVersion: v1
            kind: Pod
            ...
                  startupProbe:
                    initialDelaySeconds: 1
                    periodSeconds: 2
                    timeoutSeconds: 1
                    successThreshold: 1
                    failureThreshold: 30      <-- Number of times allowed to fail
                    httpGet:
                      scheme: HTTP
                      path: /
                      httpHeaders:
                        - name: Host
                          value: myapplication1.com
                      port: 80
          ``

- **Readiness Probe**
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
        - ``
            apiVersion: v1
            kind: Pod
            ...
            spec:
              restartPolicy: Always     # <--
              container:
                ...
          ``

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
    - ``
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
              - name: sharedvol
                emptyDir: {}
      ``


## Init Containers

Docs: https://kubernetes.io/docs/concepts/workloads/pods/init-containers/

- Containers that run during startup process
- Listed in `InitContainer` spec section
- Run before any other containers in `containers` section
- Must run to once and to completion
- Run in the order they are listed
- Used to setup the pod, install tooling, etc
- Can container tasks and setup scripts not needed in the main containers
- When starting a pod, status will read `Init` if currently running init container
- Possible use cases
    - Wait for another Kubernetes resource to be created before startup
    - Perform sensitive startup steps securely outside of app containers
    - Populate data into a shared volume at startup. Main app container can read it.
    - Communicate with another service at startup
- Example: Init container with simple startup delay
    - ``
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
      ``

- Example: Wait for a service
    -``
        apiVersion: v1
        kind: Pod
        ...
        spec:
        ...
          initContainers:
            - name: my-init-container
              image: busybox:1.28
              command: ['sh', '-c', "until nslookup my-service; do echo waiting for my-service; sleep 2; done"]
     ``



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
    - ``
        apiVersion: v1
        kind: Pod
        ...
        spec:
          ...
          nodeSelector:
            mylabel: someValue      # <-- Only nodes with this label
      ``

### `nodeName`

- Bypass scheduling logic and assign pod to a specific Node by node name
- Example:
    - ``
        ...
        spec:
          ...
          nodeName: worker0
      ``
- Getting node names
    - `kubectl get nodes`


## DaemonSets

- DaemonSets will automatically runs a copy of a Pod on each node in the cluster
- Will run copy of the Pod on new nodes as they are added to the cluster
- Manages specified pods
- Will respect scheduling rules (ie. labels, taints, tolerations)
- Belong to `apiVersion: apps/v1`
- Example: Single pod, single container in each node
    - ``
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
       ``
- Get list of DaemonSets
    - `kubectl get daemonset`



### Useful Linux Commands

- Setting Custom Hostname
    - `sudo hostnamectl set-hostname k8s-control`

- Set up host file
    - `sudo vim /etc/hosts`
    - Add private IPs for all servers
    - `<PRIVATE IP> <HOSTNAME>`








