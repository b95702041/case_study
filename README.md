# case_study

I launch 3 EC2 instance. 1 master, 2 workers.

2 workers in `t2.micro`, master in `t2.medium`,  all in` Ubuntu Server 20.04 LTS (HVM)`, SSD Volume Type.

They are in same vpc, subnet and security group.

Download private key and ssh to the instance:

```ssh -i case_study.pem ubuntu@ec2-107-22-41-11.compute-1.amazonaws.com```


Update the apt package and install packages needed to use the Kubernetes apt repository:

```
sudo apt-get update

sudo apt-get install -y apt-transport-https ca-certificates curl
```
Download the Google Cloud public signing key:

```
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
```

Add the Kubernetes apt repository:

```
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

Update apt package index, install kubelet, kubeadm and kubectl.
```
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
```
To disable swapping to avoid losing isolation properties.

```
swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

Set docker group for permission control.
```
sudo apt install docker.io -y
sudo usermod -aG docker ubuntu
sudo systemctl restart docker
sudo systemctl enable docker.service
sudo apt-get update
sudo systemctl daemon-reload
```

In master node, run the following command:
```
sudo kubeadm init --pod-network-cidr=172.31.0.0/20
```
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

now deploy a pod network to the cluster.

```
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```

Then  join 2 worker nodes by running the following on each as root:
```
sudo kubeadm join 172.31.21.216:6443 --token 87ywye.5077xl6oxp1fh6vz --discovery-token-ca-cert-hash sha256:3e4eba75d962ef774ebb1caa0d23d1cbb2a85db25747d5bf3370de63bba54fb0
```
I faced this join issue. We can port:6443 in Security Group of my AWS EC2 instance to fix this.

```
[preflight] Running pre-flight checks
	[WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/
error execution phase preflight: couldn't validate the identity of the API Server: Get "https://172.31.15.150:6443/api/v1/namespaces/kube-public/configmaps/cluster-info?timeout=10s": net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
```


Check the nodes running in your k8s cluster. This can be done with the help of the following command.
```
kubectl get nodes
```
```
kubectl get pods --all-namespaces
```


In worker node,run the following command to join k8s cluster.
```
kubeadm join --token qiey62.dbip12b1tss8thzf 172.31.18.33:6443 --discovery-token-ca-cert-hash sha256:bced5739dddcd61a168ac6f61616ccce1a7a89908813428930cc2634b1f347c0
```

Next, we create g a file index.html.

Create the config map

```
kubectl apply -f index-html-configmap.yaml
```

Now working on the deployment config which uses the index.html config map. Save the manifest as `nginx.yaml`

Create the deployment.

```
kubectl apply -f nginx.yaml
```

We create a NodePort service to access the Nginx deployment from any Kubernetes node IP on port 32000.

Save the manifest as `nginx-service.yaml`

We can access it via `worker public ip: nginx port`.
which are 107.22.41.11:32000 and 34.203.10.35:32000
I tried to use php to get nodename but didn't work in html. Will need more time to investigate.



For monitoring, if we deploy on AWS, we can use cloudwatch.
We can deploy elasticsearch and use matricbeat and filebeat for monitoring.
Also, we can collect matrics by ourselves and store in db(such as influxDB) and display in Grafana to visualize.
In addition, we can run Sensu checks on the pods. And hooked with PagerDuty and Slack for on-call engineer.


For security, if we use AWS, we can use security group, Network access control list to help manage inbound and outbound traffic.
We should be careful about the service permission and manage private key of the instance.
We can run customized checks in Tenable SecurityCenter.



`reference`:

1.Installing kubeadm

https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

2.Setting Up a Single Master Kubernetes Cluster on AWS Using Kubeadm

https://medium.com/@dileepjallipalli?p=fad03e5937cf

3.How to deploy NGINX on a Kubernetes cluster

https://www.techrepublic.com/article/how-to-deploy-nginx-on-a-kubernetes-cluster/

4.kubeadm join

https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-join/

5.Kubernetes coredns pods stuck in Pending status. Cannot start the dashboard

https://stackoverflow.com/questions/55609377/kubernetes-coredns-pods-stuck-in-pending-status-cannot-start-the-dashboard

6.Why does kubeadm not start even after disabling swap?

https://stackoverflow.com/questions/56287494/why-does-kubeadm-not-start-even-after-disabling-swap

7.kubernetes - Couldn't able to join master node - error execution phase preflight: couldn't validate the identity of the API Server

https://stackoverflow.com/questions/61305498/kubernetes-couldnt-able-to-join-master-node-error-execution-phase-preflight

8.How To Deploy NGINX on a Kubernetes Cluster

https://www.youtube.com/watch?v=HTNt6YjkZ7M

9.How to Deploy Nginx on a Kubernetes Cluster

https://www.tecmint.com/deploy-nginx-on-a-kubernetes-cluster/

10.How To Change Nginx index.html in Kubernetes With Configmap

https://scriptcrunch.com/change-nginx-index-configmap/
