Kubernetes
==========

Terms
=====
    Node: A node is a physical server or virtual server on which kubernetes is installed
    
    Cluster: A cluster is a set of Nodes grouped together
    
    Master Node: Node with kubernetes installed that is reponsible for the actual orchestration.  
    It contains the: API server, ETCD, Controller, Scheduler

    Worker Node: Node with kubernetes installed that is responsible for running containers  
    It Contains: kubelet, container runtime

What happens when you install kubernetes on a server
----------------------------------------------------
You install:
API: Front end for Kubernetes; The users, management devices, command line interfaces all talk to the API server to interact with Kubernetes cluster
ETCD service: Distributed key-value store used by Kubernetes to store all the data used to manage the cluster. Also responsible for using locks to avoid conflicts between masters
kubelet service: 
    Agent that runs on each node in the cluster. 
    Reponsible for making sure that the containers are running on the nodes as expected
    Provide health information of the worker node
    Carry out actions requested by the master node on the worker nodes
Container Runtime: Underlying software that is used to run containers (Docker in our case. rkt and CRI-O are other options)
Container Controllers: Reponsible for noticing and responding when nodes, containers and endpoints go down. The controllers make decisions to bring up new containers in such cases.
Container Scheduler: Responsible for distributing the work or containers across multiple nodes

What is a pod
-------------
A pod is a single instance of an application
Smallest object that you can create in Kubernetes
A single pod can have multiple containers but should not have multiple containers of the same kind
Pods share the same network so each container in each pod can communicate to each other through localhost
Pods share the same storage
Multi-container pods are a rare usecase  

kubectl commands
================
kubectl cluster-info: view information about the cluster

kubectl run
-----------
kubectl run hello-minikube: Used to deploy an application on the cluster
kubectl run nginx --image nginx: parameter used to specify image to use (By default uses dockerhub)

Kubectl get
-------------------
kubectl get nodes: List all the nodes of the cluster
kubectl get pods: List all the pods of the cluster
kubectl get pod <pod-name> -o yaml > pod-definition.yaml: Extract pod definition file
kubectl get replicaset: List all the replicaset of the cluster
kubectl get deployments: List all the deployments of the cluster
kubectl get configmaps: Lists all the configmaps of the cluster
kubectl get secrets: List all the secrets of the cluster
kubectl get secrets <secret-name> -o yaml: View hashed value
kubectl get serviceaccount: View all service accounts
kubectl get all: Get deployments, replicaset and pods
kubectl get all --all-namespaces: Get deployments, replicaset and pods in all namespaces
kubectl get <objects> --namespace <namespace-name>: gets object in the namespace named namespace-name 

kubectl create
--------------
kubectl create -f pod-definition.yml: creates a pod following a definition object
kubectl create -f replicaset-definition.yml: creates a replicaset following a definition object
kubectl create -f deployment-definition.yml: creates a deployment following a definition object
kubectl create -f object-definition.yml --namespace <namespace-name>: creates pod in a different namespace

kubectl create serviceaccount
-----------------------------
kubectl create serviceaccount <account-name>


kubectl describe
----------------
kubectl describe pod <pod-name>: detailed information about the pod
kubectl describe secrets: detailed information about the secrets in the cluster
kubectl describe serviceaccount <service-account-name>: gets information about the service account
kubectl describe secret <service-account-token-name>: gets the token of the service account

kubectl delete
--------------
kubectl delete pod webapp: delete pod named webapp
kubectl delete deployment nginx: delete pod named nginx

kubectl edit
------------
kubectl edit pod <pod-name>: edit pod properties

kubectl replace
---------------
kubectl replace -f replicaset-definition.yml (Changes file)

kubectl scale
-------------
kubectl scale --replicas=6 -f replicaset-definition.yml (Equivalent to kubectl replace -f replicaset-definition.yml (but no change to definition)
kubectl scale --replicas=6 replicaset myapp-replicaset (Equivalent to kubectl replace -f replicaset-definition.yml (but no change to definition)

kubectl config
--------------
kubectl config current-context: get current namespace context
kubectl config set-context $(kubectl config current-context) --namespace dev: change current namespace context to dev

kubectl exec
------------
kubectl exec -it <pod-name> ls /: launches ls command on pod at /

Formating
---------
The default output format for all kubectl commands is the human readable plain-text format.

The -o flag allows to output the details in several formats
kubectl [command] [type] [name] -o <output-format>

1. -o jsonOutput a JSON formatted API object.
2. -o namePrint only the resource name and nothing else.
3. -o wideOutput in the plain-text format with any additional information.
4. -o yamlOutput a YAML formatted API object.

For more details, refer:
https://kubernetes.io/docs/reference/kubectl/overview/
https://kubernetes.io/docs/reference/kubectl/cheatsheet/

dry-run
-------
--dry-run: default parameter, as soon as the command is run, the resource will be created
--dry-run=client: does not create the resource, only tells you wheather the resource could be created or not


Kubernetes definition file
--------------------------
A kubernetes definition file always contains 4 top level fields (They are required):
    apiVersion:
    kind:
    metadata:
    spec:

Pod definition file
-------------------
apiVersion: Version of the kubernetes API to create the Object (example: v1, apps/v1, apps/v1)
kind: Type of object we are trying to create (example: Pod, ReplicationController, ReplicaSet, Service, Deployment)
metadata: data about the object (dictionary) (You cannot add any key-value under metadata):
    name: (string)
    labels: (dictionary) (You can add any label)
        app: (string)
spec: Specification function (dictionary) (specific for each kind)
    containers:
        - name: nginx-container
          image: nginx

Replication Controller
----------------------
The replication controller helps us run multiple instances of a Pod

ReplicationController definition
--------------------------------
apiVersion: v1
kind: ReplicationController
metadata:
    name:
    labels:
        app:
spec: (dictionary)
    template:
        metadata:
            name:
            labels:
        spec:
            containers:
            -  name: nginx-container
               image: nginx
    replicas: 3

ReplicaSet
----------
Newer version of ReplicationController

The major difference between ReplicaSet and ReplicationController is the selector param in the ReplicaSet Definition

If it is not specified it wil assume the labels of the template to be the selector

ReplicaSet definition
---------------------
apiVersion: apps/v1
kind: ReplicaSet
metadata:
    name: myapp-replicaset
    labels:
        app: myapp
        type: front-end
spec: (dictionary)
    template:
        metadata:
            name:
            labels:
        spec:
            containers:
            -  name: nginx-container
               image: nginx
    replicas: 3
    selector:
        matchLabels:
            type: front-end

IMPORTANT NOTE: ReplicaSet Ensure that the number of pods are always running. So deleting one will just make it go back up again.

TODO: matchLabels what are they used for ?

Deployment
==========
Higher object in the hiarchy than replicaset and pod

It provides the capability to upgrade the underlying instances seamlessly using:
    rolling updates
    undo changes
    pause
    resume changes

Deployment definition
---------------------
Same as replicaset except for the kind which is **Deployment**

apiVersion: apps/v1
kind: Deployment
metadata:
    name: myapp-replicaset
    labels:
        app: myapp
        type: front-end
spec: (dictionary)
    template:
        metadata:
            name:
            labels:
        spec:
            containers:
            -  name: nginx-container
               image: nginx
    replicas: 3
    selector:
        matchLabels:
            type: front-end

Namespaces
----------
Namespaces can refer to objects within their own namespaces by their name

To refer to objects outside of their own namespaces, objects should use their full name.

Example: name.namespace.svc.cluster.local

cluster.local: is the default domain of the kubernetes cluster

svc: is the subdomain of the service

namespace: is the namespace

name: is the same of the objct

Get commands only list objects in the default namespace
To list pods in a different namespace add the  --namespace flag folled by the name of the namespace


Kube creates by default your default namespace, kube-system namespace and kube-public namespace

kube-system
-----------
contains the set of pods and services used for internal purposes, such as the networking solution, dns etc..

This is done to prevent accidental deletion by the user


kube-public
-----------
This is where resources that should be made available to all users are created

Resource quota
--------------
You can limit resources in a namespace by creating a resource quota

Resource quota definition file
------------------------------
apiVersion: v1
kind: ResourceQuota
metadata:
    name: compute-quota
    namespace: dev
spec:
    hard:
        pods: "10"
        requests.cpu: "4"
        requests.memory: 5Gi
        limits.cpu: "10"
        limits.memory: 10Gi

Contexts
--------
Used to manage multiple clusters in multiple environments from the same management system.

Imperative commands
-------------------
Commands used to generate configuration files fast

Pod:
    kubectl run nginx --image=nginx --dry-run=client -o yaml

Deployment
    kubectl create deployment --image=nginx --dry-run=client -o yaml

Generate Deployment with 4 Replicas
    kubectl create deployment nginx --image=nginx --replicas=4

Scale a kubernetes deployment
    kubectl scale deployment nginx --replicas=4

Environment variables
---------------------
To declare environment variables add the env key to your container in the spec

For ConfigurationMaps just add the envFrom key

Example:
```yaml
apiVersion: v1
kind: Pod
metadata:
    name: my-pod
spec:
    containers:
    - name: my-container
      image:
      env:
      - key: KEY1
        value: value1
      - key: KEY2
        value: value2
      envFrom:
        - configMapRef:
            name: <config-map-name>
```

ConfigMaps
----------
Allows to manage environment variables in centralized environment.

Config maps allow to pass configuration data in the form of key value pairs in Kubernetes

When a pod is created you can inject the config map into the pod

To inject a config map just add the key envFrom to the container you want to associate the configmap to:

Example
```yaml
apiVersion: v1
kind: Pod
metadata:
    name: my-pod
spec:
    containers:
    - name: my-container
      image:
      envFrom:
      -  configMapRef:
            name: app-configuration
```

How to create a ConfigMap
-------------------------
Imperative way
--------------
First way:
```bash
kubectl create configmap \
    <config-name> --from-literal=APP_COLOR=blue
                  --from-literal=APP_MOD=prod
```

Second way:
path-to-file:
```
APP_COLOR: blue
APP_MODE: prod
```
```bash
kubectl create configmap \
    <config-name> --from-file=<path-to-file>
```

Declarative way
1. Create a definiton file
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
    name: app-config
data:
    APP_COLOR: blue
    APP_MODE: prod
```
Then run the command: *kubectl create -f*

Secrets
=======
To create a secret you have to first create it and then inject it into a Pod

Creating a secret
-----------------
Imperative way:
---------------
```yaml
Generic:
kubectl create secret generic \
    <secret-name> --from-literal=<key>=<value>

Example:
kubectl create secret generic \
    <secret-name> --from-literal=DB_HOST=mysql \
                  --from-literal=DB_USER=root \
                  --from-literal=DB_Password=paswrd

Generic using a file:
kubectl create secret generic \
    <secret-name> --from-file=<path-to-file>

Example using a file:
File:
DB_HOST: mysql
DB_USER: root
DB_Password: paswrd

Example:
kubectl create secret generic \
    app-secret --from-file=app_secret.properties
```

Declarative way
---------------
We use a definition file

```yaml
apiVersion: v1
kind: Secret
metadata:
    name: app-secret
data: # Encode values here using the base64 utility in linux
    DB_HOST: mysql
    DB_USER: root
    DB_Password: paswrd
```

Encoding values in linux
```bash
echo -n 'base64value' | base64
```

Decoding values in linux
```bash
echo -n 'base64value' | base64 --decode
```

Injecting secrets in pods
-------------------------
pod definition yaml file
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-name
spec:
  containers:
  - name: simple-web-app
    envFrom: # First way
      - secretRef:
          name: app-secret
    
    env: # Second way
      - name: DB_PASSWORD
        valueFrom:
          secretKeyRef:
            name: app-secret
            key: DB_PASSWORD

    volumes: # Third way
      - name: app-secret-volume
        secret: 
          secretName: app-secret
```

## Questions to answer
What is the type of a secret ?



Exam tips
=========
A Note on Editing Existing Pods
-------------------------------
In any of the practical quizzes if you are asked to edit an existing POD, please note the following:
    If you are given a pod definition file, edit that file and use it to create a new pod.
    If you are not given a pod definition file, you may extract the definition to a file using the below command:
    kubectl get pod <pod-name> -o yaml > pod-definition.yaml
    Then edit the file to make the necessary changes, delete and re-create the pod.
    Use the kubectl edit pod <pod-name> command to edit pod properties.

Docker Security
---------------
/usr/include/linux/capability.h: File where is stored the full list of capabilities of a root user.

On docker, a root user does not have the same capabilities than a root user on a machine.
For example, a root user running on a docker host cannot turn off a vm or change network settings.

To override this behavior, you can use the --cap-add option to add capabilities

To remove capabilities add the --cap-drop option

To add all privileges, add the --privileged option

Kubernetes security Context
===========================
You can configure the security setting at a container level or at a pod level.

Configuring at the pod level makes security settings carry over to all containers within the pod

Settings on containers override settings on the Pod

Configuring security context in a pod
-------------------------------------
Example:
```yaml
apiVersion: v1
kind: Pod
metadata:
    name: web-pod
spec:
    securityContext:
        runAsUser: 1000

    containers:
        - name: ubuntu
          image: ubuntu
          command: ["sleep", "3600"]
```

Configuring security context in a container
-------------------------------------
Example:
```yaml
apiVersion: v1
kind: Pod
metadata:
    name: web-pod
spec:

    containers:
        - name: ubuntu
          image: ubuntu
          command: ["sleep", "3600"]
          securityContext:
              runAsUser: 1000
              capabilities:
                add: ["MAC_ADMIN"]
```
Service Accounts
================
Account used by an application to access the kubernetes cluster

kubectl describe serviceaccount <service-account-name>: gets information about the service account

Service account token is used to let an external application authenticate to kubernetes

You can get the service account *token* from the previous command

The token is stored in a secret. To retrieve the token launch the following command:  
kubectl describe secret <service-account-token-name>: gets the token of the service account

If you have an application running on the kubernetes cluster that wants to authenticate to kubernetes, you can mount the service token secret as a volume inside the pod hosting the third party application.

There is a service account for each namespace by default

Whenever a pod is created, the service account and its token are automatically mounted to that pod as a volume mount
You can see that by using the kubectl describe pod <pod-name>

The default service account has limited permissions (it has permissions to only run basic kube api queries)

You cannot edit the service account of an existing pod, it must be deleted and recreated.

For deployments you can edit the service account as it would automatically trigger a new rollout

To add service account to a container:

```yaml
spec:
    containers:
        serviceAccount: <service-account-name>
```

To avoid automounting the default service account set:
```yaml
spec:
    containers:
        automountServiceAccountToken: false
```
