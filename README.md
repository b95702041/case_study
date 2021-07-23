# case_study

I launch 3 EC2 instance. 1 master, 2 workers.

2 workers in `t2.micro` master in `t2.medium`, with Ubuntu Server 20.04 LTS (HVM), SSD Volume Type.

They are in same vpc, subnet and security group.

```ssh -i case_study.pem ubuntu@ec2-107-22-41-11.compute-1.amazonaws.com```


Update the apt package index and install packages needed to use the Kubernetes apt repository:

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

Update apt package index, install kubelet, kubeadm and kubectl, and pin their version:
```
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
```
```
swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
sudo apt install docker.io -y
sudo usermod -aG docker ubuntu
sudo systemctl restart docker
sudo systemctl enable docker.service
sudo apt-get update
sudo systemctl daemon-reload
```

Kubernetes_master

in master

sudo kubeadm init --pod-network-cidr=172.31.0.0/20

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.

kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"


Then you can join any number of worker nodes by running the following on each as root:
```
sudo kubeadm join 172.31.21.216:6443 --token 87ywye.5077xl6oxp1fh6vz --discovery-token-ca-cert-hash sha256:3e4eba75d962ef774ebb1caa0d23d1cbb2a85db25747d5bf3370de63bba54fb0
```

```
[preflight] Running pre-flight checks
	[WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/
error execution phase preflight: couldn't validate the identity of the API Server: Get "https://172.31.15.150:6443/api/v1/namespaces/kube-public/configmaps/cluster-info?timeout=10s": net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
```
Simple fix: Expose port:6443 in Security Group of my AWS EC2 instance.


Check the nodes running in your k8s cluster. This can be done with the help of the following command.
```
kubectl get nodes
```
```
kubectl get pods --all-namespaces
```



in worker node
```
kubeadm join --token qiey62.dbip12b1tss8thzf 172.31.18.33:6443 --discovery-token-ca-cert-hash sha256:bced5739dddcd61a168ac6f61616ccce1a7a89908813428930cc2634b1f347c0
```

in master



worker public ip: nginx port
which are 107.22.41.11:32000 and 34.203.10.35:32000


sudo apt install php7.4-cli

cat index-html-configmap.yaml




For monitoring, if we deploy on AWS, we can use cloudwatch.
We can also deploy elastic and use matricbeat and filebeat for monitoring pod.
we can also collect matrics by ourslef and store in db(such as influxDB) and monitor in Grafana visulization tool.

For security, if we use AWS, we can use security group, Network access control list to help manage inbound and outbound traffic.
We should be careful about the service permission. And manage private key to the instance.


reference:

1.https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

2.https://medium.com/@dileepjallipalli?p=fad03e5937cf

3.https://www.techrepublic.com/article/how-to-deploy-nginx-on-a-kubernetes-cluster/

4.https://www.geeksforgeeks.org/get-post-requests-using-python/

5.https://techoverflow.net/2019/03/31/how-to-install-kubectl-on-ubuntu/

6.https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-join/

7.https://stackoverflow.com/questions/55609377/kubernetes-coredns-pods-stuck-in-pending-status-cannot-start-the-dashboard

8.https://stackoverflow.com/questions/56287494/why-does-kubeadm-not-start-even-after-disabling-swap

9.https://stackoverflow.com/questions/61305498/kubernetes-couldnt-able-to-join-master-node-error-execution-phase-preflight

10.https://www.youtube.com/watch?v=HTNt6YjkZ7M

11.https://www.tecmint.com/deploy-nginx-on-a-kubernetes-cluster/

12.https://scriptcrunch.com/change-nginx-index-configmap/
