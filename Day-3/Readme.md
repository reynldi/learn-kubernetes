## Kubernetes API Primitives

Kubernetes API primitive, also known as Kubernetes objects, are the basic building blocks of any application running in Kubernetes. Building and managing Kubernetes applications means working with objects. It is essential to be familiar with Kubernetes objects in order to develop applications for Kubernetes

Here is sample of Kubernetes Objects

- Pod
- Node
- Service
- Service Account

We can see list of Kubernetes API Primitives or Kubernetes object by executing this command: <br/>
`kubectl api-resources`

Relevant Documentation about kubernetes object
https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/

Every object in kubernetes cluster has 2 basic global property:

1. **Spec** : User will define/provide the desire state / spesification of object
2. **Status** : This is provided by kubernetes cluster server to inform about current status of kubernetes object

Sample execute one of kubernetes object with this command: <br/>

```
kubectl api-resources -o name

kubectl get pods -n kube-system

kubectl get nodes

kubectl get nodes $node_name

kubectl get nodes $node_name -o yaml

kubectl describe node $node_name
```

## Indept with Pods

Pods are one of the most essential Kubernetes object types. Most of the orchestration features of Kubernetes are centered around the management of Pods. Pods consist of one container or more than one containers. <br/>

Relevant Documentation <br/>
https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/

- Create a new yaml file to contain the pod definition

      	apiVersion: v1
      	kind: Pod
      	metadata:
      	name: my-pod
      	labels:
      		app: myapp
      	spec:
      	containers:
      	- name: myapp-container
      		image: busybox
      		command: ['sh', '-c', 'echo Hello Kubernetes! && sleep 3600']

- Every object has API version, on this case we use v1 as API version. and then we define **kind**, **kind** is type of object in kubernetes. Each object that we define has **metadata**. All kubernetes object has spesification, we will define specification on **spec** section on that yanl file.

- Create a pod from the yaml definition file: <br/>
  `kubectl create -f my-pod.yml`

- Edit a pod by updating the yaml definiton and re-applying it: <br/>
  `kubectl apply -f my-pod.yml`

- You can also edit a pod like this: <br/>
  `kubectl edit pod my-pod`

- You can delete a pod like this: <br/>
  `kubectl delete pod my-pod`

## Indept with Namespaces

Namespaces provide a way to keep your object organized well in your kubernetes cluster. We can group our kubernetes object into Namespaces. If we are not define Namespaces, kubernetes will use `default` namespaces.

We can define namespace into our kubernetes object when we create that object <br/>
Relevant Documentation
https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/ <br/>

- You can get a list of the namespaces in the cluster like this:<br/>
  `kubectl get namespaces`
- You can also create your own namespaces.<br/>
  `kubectl create ns my-namespace`
- We can edit our last yaml file and define our name space

      	apiVersion: v1
      	kind: Pod
      	metadata:
      	name: my-ns-pod
      	namespace: my-namespace
      	labels:
      		app: myapp
      	spec:
      	containers:
      	- name: myapp-container
      		image: busybox
      		command: ['sh', '-c', 'echo Hello Kubernetes! && sleep 3600']

- After that we can re-create again the pods and trying to get pods: <br/>
  `kubectl get pods`
- You will never see the new pods, because that command will only get pods in `default` pods.
- So we need to define namespace when execute get pods command.<br/>
  `kubectl get pods -n my-namespace`
