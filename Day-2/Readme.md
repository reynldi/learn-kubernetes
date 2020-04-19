## Notes
- Pods are the smalles unit and most basic bulding block in Kubernetes

- A pods can have one or more containers, storage and any resources also has unque IP address

- The server which actually run on our containers are called nodes

- Nodes are an essential part of the Kubernetes cluster. They are the machines where your cluster's container workloads are executed

- A kubernetes cluster has one or more control server ( Master server or we called it Kube Master )

- You can execue this command to get useful information about  node
	`kubectl get nodes` and `kubectl describe node $node_name`

## Kubernetes Components
- **etcd** : Provide distributed and syncronized data storage for cluster. It has syncronized all data storage between nodes. So it will make sure if all data in the state all same

- **kube-apiserver** : Serve kubernetes API, the primary interfaces for the cluster

- **kube-scheduler** : Schedules pods to run on individual nodes

- **kubelet** : Agent that executes container on each node.

- **kube-proxy** : Handle networks communication between nodes by adding firewall routing rules.

You can see some of kubernetes components by executing `kubectl get pods -n kube-system`. There's no `kubelet` because `kubelet` is running as OS service in each node. You can execute this command to see `kubelet` status `sudo systemctl status kubelet`.

## Create Simple Pods
- Create a simple pod running an nginx container:
	```
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
- Get a list of pods and verify that your new nginx pod is in the Running state <br/>
	`kubectl get pods`

- You can get all system pods by running  <br/>
	`kubectl get pods -n kube-system`  <br/>
	`-n` that's mean name space

- Get more information about your nginx pod: <br/>
	`kubectl describe pod nginx`

- Delete the pod: <br/>
	`kubectl delete pod nginx`

## Networking in Kubernetes
- Previously we already setup our master networking with Flunnel plugin. So each pods on different worker nodes they can make communication each other. We just need to know the IP address each of pods.

- Create a deployment with two nginx pods:
	```
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

- Create a busybox pod to use for testing:
	```
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

- Get the IP addresses of your pods: <br/>
	`kubectl get pods -o wide`

- Get the IP address of one of the nginx pods, then contact that nginx pod from the busybox pod using the nginx pod's IP address: <br/>
	`kubectl exec busybox -- curl $nginx_pod_ip`

## Kubernetes Deployments
Deployments are an important tool if you want to take full advantage of the automation capabilities provided by Kubernetes. It has responsibillity to handle your pods if in case your pods are crash, deployment will re-create another pods. Deployment also can do automatic scalling up or scalling down base on state desire. You can configure deployment in yaml file.

- Create a deployment:
	```
	cat <<EOF | kubectl create -f -
	apiVersion: apps/v1
	kind: Deployment
	metadata:
	name: nginx-deployment
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

- Get a list of deployments: <br/>
`kubectl get deployments`

- Get more information about a deployment: <br/> 
`kubectl describe deployment nginx-deployment`

- Get a list of pods: <br/>
`kubectl get pods`

- You should see two pods from deployment. If you try to delete on of them with this command `kubectl delete pods $pods_name` deployment will handle it to re-create another pods as fast as they can.

## Kubernetes Services
While deployments provide a great way to automate the management of your pods, you need a way to easily communicate with the dynamic set of replicas managed by a deployment. This is where services come in.

Services allow you to to dynamically access a group of replica pods. Because the pods are dynamic when we scalling the pods or deleting the pods, service will handle it. Service will routing and willd decide to route your request into on of available pods.

Here are the commands used in the demonstration: <br/>
- Create a NodePort service on top of your nginx pods:
```
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

- Get a list of services in the cluster. <br/>
`kubectl get svc` <br/>
You should see your service called `nginx-service`

- Since this is a NodePort service, you should be able to access it using port 30080 on any of your cluster's servers. You can test this with the command: <br/>
`curl localhost:30080`

- You should get an HTML response from nginx!


