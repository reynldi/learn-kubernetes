## Notes
- What we are doing in Master cluster, we also need to the same thing in node cluster
- There's three main component of kubernetes:
	- **Kubeadm**: Is the tool that helps you bootstrap a minimum viable Kubernetes cluster that conforms to best practices. It will make our job more easier to setup large portion of cluster setup.

	- **Kubelet**: The essential component of kubernetes that handle running container on nodes. Every server needs kubelet. It will handles how to spin up container, how to tear down contaier.etc. They will manage individual node in general.

	- **Kubectl**: is command-line tool to interact with our cluster. so it will easier for us to manage the cluster.

## Initialize Node
- On the Kube master node, initialize the cluster: <br/>
	`sudo kubeadm init --pod-network-cidr=10.244.0.0/16` <br/>
	That command may take a few minutes to complete.

- When it is done, set up the local kubeconfig: <br/>
	```
	mkdir -p $HOME/.kube
	sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
	sudo chown $(id -u):$(id -g) $HOME/.kube/config
	```

- Verify that the cluster is responsive and that Kubectl is working: <br/>
	`kubectl version`

- You should get `Server Version` as well as `Client Version`. It should look something like this: <br/>
	```
	Client Version: version.Info{Major:"1", Minor:"12", GitVersion:"v1.12.2", GitCommit:"17c77c7898218073f14c8d573582e8d2313dc740", GitTreeState:"clean", BuildDate:"2018-10-24T06:54:59Z", GoVersion:"go1.10.4", Compiler:"gc", Platform:"linux/amd64"}
	Server Version: version.Info{Major:"1", Minor:"12", GitVersion:"v1.12.2", GitCommit:"17c77c7898218073f14c8d573582e8d2313dc740", GitTreeState:"clean", BuildDate:"2018-10-24T06:43:59Z", GoVersion:"go1.10.4", Compiler:"gc", Platform:"linux/amd64"}
	```
- The `kubeadm init` at the first time ( point number 1) command should output a `kubeadm join` command containing a token and hash. Copy that command and run it with sudo on both worker nodes ( node 1 and node 2). It should look something like this: <br/>
`sudo kubeadm join $some_ip:6443 --token $some_token --discovery-token-ca-cert-hash $some_hash
`

- Verify that all nodes have successfully joined the cluster. <br/>
`kubectl get nodes` <br/>
You should see all three of your nodes listed. It should look something like this: <br/>
	```
	NAME                      STATUS     ROLES    AGE     VERSION
	wboyd1c.mylabserver.com   NotReady   master   5m17s   v1.12.2
	wboyd2c.mylabserver.com   NotReady   <none>   53s     v1.12.2
	wboyd3c.mylabserver.com   NotReady   <none>   31s     v1.12.2
	```

## Configuring Networking with Flannel
Actually Kubernetes supports variety of different networks of plugins that can implement in kubernetes networking. in this case we will use Flannel plugin to setup our kubernetes networking. You can find more information on Flannel at the official site: https://coreos.com/flannel/docs/latest/.

- On all three nodes, run the following command:
	```
	echo "net.bridge.bridge-nf-call-iptables=1" | sudo tee -a /etc/sysctl.conf \
	sudo sysctl -p
	```
- Install flannel plugin on `Master Cluster` only with this command <br/>
	```
	kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/bc79dd1505b0c8681ece4de4c0d86c5cd2643275/Documentation/kube-flannel.yml
	```
- Now if you execute `kubectl get nodes` as you can see all nodes on `Ready` state
	```
	NAME                      STATUS     ROLES    AGE     VERSION
	wboyd1c.mylabserver.com   Ready      master   5m17s   v1.12.2
	wboyd2c.mylabserver.com   Ready      <none>   53s     v1.12.2
	wboyd3c.mylabserver.com   Ready      <none>   31s     v1.12.2
	```
- It is also a good idea to verify that the Flannel pods are up and running. Run this command to get a list of system pods: <br/>
`kubectl get pods -n kube-system` <br/>
You should have three pods with flannel in the name, and all three should have a status of Running.
## Day 1 Learn Checklist:
- [x] Kubernetes Fundamental
- [x] Setup Three Nodes ( Kubernetes Master, Kube Node 1, Kube Node 2)
- [x] Setup Kubernetes tools ( kubeadm, kubelet, kubectl ) 