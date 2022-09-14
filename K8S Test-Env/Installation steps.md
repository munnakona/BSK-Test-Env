INSTALLATION OF KUBERNETES CLUSTER STEPS
---------------------------------------------


STEPS: INSTALLATION OF DOCKER ENGINE ON ALL NODES:
------------------------------------------------------

curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh
sudo -i
# Run these commands as root :
------------------------------------------------

###Install GO###
wget https://storage.googleapis.com/golang/getgo/installer_linux
chmod +x ./installer_linux
./installer_linux
source ~/.bash_profile
git clone https://github.com/Mirantis/cri-dockerd.git
cd cri-dockerd
mkdir bin
go get && go build -o bin/cri-dockerd
mkdir -p /usr/local/bin
install -o root -g root -m 0755 bin/cri-dockerd /usr/local/bin/cri-dockerd
cp -a packaging/systemd/* /etc/systemd/system
sed -i -e 's,/usr/bin/cri-dockerd,/usr/local/bin/cri-dockerd,' /etc/systemd/system/cri-docker.service
systemctl daemon-reload
systemctl enable cri-docker.service
systemctl enable --now cri-docker.socket
usermod -aG docker ubuntu
sudo usermod -aG sudo kubeadmin
sudo systemctl start docker && sudo systemctl enable docker 

cat <<EOF | sudo tee /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF


sudo systemctl daemon-reload && sudo systemctl restart docker


INSTALL KUBELET,KUBEADM,KUBECTL IN ALL NODES:
---------------------------------------------

sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl

sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
sudo swapoff -a

Kubeadm Setup:
------------------

kubeadm init --pod-network-cidr=10.244.0.0/16 --cri-socket unix:///var/run/cri-dockerd.sock

OUTPUT:
-------------

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

 To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 172.31.30.97:6443 --token 1au53h.3tvngy9k68j1utnu \
        --discovery-token-ca-cert-hash sha256:8cb5d596699989c3419766770de24a114dd2b60a8c482fde3b26c9958536c36b --cri-socket unix:///var/run/cri-dockerd.sock
-----------------------------------------------------------------------------------------------------
exit from root user

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config



Kube Network Setup using Flanel:
------------------------------------

kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml

kubectl get nodes -w

kubectl get pods --all-namespaces


Now login into the node 1 and become a root user and then execute:
-------------------------------------------------------------------
kubeadm join 172.31.9.186:6443 --token oc7nyd.g36rl6lycyk15ywl \
        --discovery-token-ca-cert-hash sha256:ef3f2b626d6168b4d2cbb64a044285def92423aa312f37405d49adb31a356815





        Now perform the same on the node 2 by executing the same command shown below as a root user:
--------------------------------------------------------

kubeadm join 172.31.9.186:6443 --token oc7nyd.g36rl6lycyk15ywl \
        --discovery-token-ca-cert-hash sha256:ef3f2b626d6168b4d2cbb64a044285def92423aa312f37405d49adb31a356815


        Now login into master/control plane and execute :
        -----------------------------------------------------
        
        kubectl get nodes

kubectl autocomplete:
---------------------------

source <(kubectl completion bash) # setup autocomplete in bash into the current shell, bash-completion package should be installed first.
echo "source <(kubectl completion bash)" >> ~/.bashrc # add autocomplete permanently to your bash shell.

This is official k8s client:
-------------------------------
Checking cluster status kubectl version

To check whether your cluster is healthy:
-------------------------------------------
 kubectl get componentstatuses


kubectl has two primary commands to obtain information:
----------------------------------------------------------
get
describe


1) kubectl get nodes
2)kubectl describe nodes ip-172-31-9-120
3)kubectl get nodes -o wide




kube-proxy:
---------------------
kube proxy is responsibe for routing network traffic in the k8s cluster. To do this job, the proxy should be present on all the nodes in the cluster


kubectl get daemonsets --namespace=kube-system kube-proxy


kuberenetes DNS :
------------------------
kuberentes also runs a DNS server, which provides naming and discovery for the services in k8s cluster

kubectl get deployments --namespace=kube-system coredns


There is also a k8s service that performs load balancing for the DNS server:
---------------------------------------------------------

kubectl get services --namespace=kube-system


