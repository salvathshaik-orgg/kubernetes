# Install Kubernetes Using Script

### `Step1: On Master Node Only`
```
##remove previous docker dependencies, Below is for Centos/oracle. please check online for other distributions
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine \
                  podman \
                  runc

## Install Docker
##sudo wget https://raw.githubusercontent.com/lerndevops/labs/master/scripts/installDocker.sh -P /tmp
sudo wget https://raw.githubusercontent.com/salwad-basha-shaik/labs/master/scripts/installDocker.sh -P /tmp
sudo chmod 755 /tmp/installDocker.sh
sudo bash /tmp/installDocker.sh
sudo systemctl restart docker.service
sudo systemctl enable docker.service
#wait for 15 seconds and validate below
sleep 15
sudo systemctl status docker.service

## Install CRI-Docker
#sudo wget https://raw.githubusercontent.com/lerndevops/labs/master/scripts/installCRIDockerd.sh -P /tmp
sudo wget https://raw.githubusercontent.com/salwad-basha-shaik/labs/master/scripts/installCRIDockerd.sh -P /tmp
sudo chmod 755 /tmp/installCRIDockerd.sh
sudo bash /tmp/installCRIDockerd.sh
sudo systemctl restart cri-docker.service
sudo systemctl enable cri-docker.service
#wait for 15 seconds and validate below
sleep 15
sudo systemctl status cri-docker.service

##Need to comment plugin section in the file etc/containerd/config.toml
##Below is for centos/oracle. please check online for other distributions
sudo sed -i 's/^disabled_plugins = \["cri"\]/#&/' /etc/containerd/config.toml
systemctl restart containerd
systemctl status containerd


##Need to stop the swap, otherwise, kubelet will not run, Below is for Centos/oracle. please check online for other distributions
swapoff -a

## Install kubeadm,kubelet,kubectl , for oracle or centos
#sudo wget https://raw.githubusercontent.com/lerndevops/labs/master/scripts/installK8S.sh -P /tmp
sudo wget https://raw.githubusercontent.com/salwad-basha-shaik/labs/master/scripts/installK8S.sh -P /tmp
sudo chmod 755 /tmp/installK8S.sh
sudo bash /tmp/installK8S.sh
#wait for 15 seconds and validate below
##sleep 15
##sudo systemctl status kubelet
kubelet --version
kubeadm version
kubectl version --client

## Install kubernetes for ubuntu
#follow this link: https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/#install-using-native-package-management

sudo apt-get update
# apt-transport-https may be a dummy package; if so, you can skip that package
sudo apt-get install -y apt-transport-https ca-certificates curl gnupg

# If the folder `/etc/apt/keyrings` does not exist, it should be created before the curl command, read the note below.
# sudo mkdir -p -m 755 /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.33/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
sudo chmod 644 /etc/apt/keyrings/kubernetes-apt-keyring.gpg # allow unprivileged APT programs to read this keyring

# Add the appropriate Kubernetes apt repository. If you want to use Kubernetes version different than v1.33,replace v1.33 with the desired minor version in the command below:
# This overwrites any existing configuration in /etc/apt/sources.list.d/kubernetes.list
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.33/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo chmod 644 /etc/apt/sources.list.d/kubernetes.list   # helps tools such as command-not-found to work correctly

# Note:
# To upgrade kubectl to another minor release, you'll need to bump the version in /etc/apt/sources.list.d/kubernetes.list before running apt-get update and apt-get upgrade. This procedure is described in more detail in Changing The Kubernetes Package Repository.

#Update apt package index, then install kubectl:
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo systemctl status kubelet
kubelet --version
kubeadm version
kubectl version --client
#sudo apt-mark hold kubelet kubeadm kubectl



## Initialize kubernetes Master Node
 
   sudo kubeadm init --cri-socket unix:///var/run/cri-dockerd.sock --ignore-preflight-errors=all

   sudo mkdir -p $HOME/.kube
   sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
   sudo chown $(id -u):$(id -g) $HOME/.kube/config

   ## install networking driver -- Weave/flannel/canal/calico etc... 

   ## below installs calico networking driver 
    
   ##kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.24.1/manifests/calico.yaml

    sudo wget https://docs.projectcalico.org/manifests/calico.yaml -P /tmp
    sudo sed -i 's|docker.io/calico|quay.io/calico|g' /tmp/calico.yaml
    kubectl apply -f /tmp/calico.yaml

   ##Run below two commands if you want to have single node cluster
	    #mnodename=`kubectl get nodes | grep control-plane | awk '{print $1}'`
      #kubectl taint node $mnodename node-role.kubernetes.io/control-plane:NoSchedule-

    #Wait for 2minutes and validate
    # Validate:  kubectl get nodes
```

### `Step2: On All Worker Nodes`

```
##remove previous docker dependencies, Below is for Centos/oracle. please check online for other distributions
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine \
                  podman \
                  runc

## Install Docker
##sudo wget https://raw.githubusercontent.com/lerndevops/labs/master/scripts/installDocker.sh -P /tmp
sudo wget https://raw.githubusercontent.com/salwad-basha-shaik/labs/master/scripts/installDocker.sh -P /tmp
sudo chmod 755 /tmp/installDocker.sh
sudo bash /tmp/installDocker.sh
sudo systemctl restart docker.service
sudo systemctl enable docker.service
#wait for 15 seconds and validate below
sleep 15
sudo systemctl status docker.service

## Install CRI-Docker
#sudo wget https://raw.githubusercontent.com/lerndevops/labs/master/scripts/installCRIDockerd.sh -P /tmp
sudo wget https://raw.githubusercontent.com/salwad-basha-shaik/labs/master/scripts/installCRIDockerd.sh -P /tmp
sudo chmod 755 /tmp/installCRIDockerd.sh
sudo bash /tmp/installCRIDockerd.sh
sudo systemctl restart cri-docker.service
sudo systemctl enable cri-docker.service
#wait for 15 seconds and validate below
sleep 15
sudo systemctl status cri-docker.service

##Need to comment plugin section in the file etc/containerd/config.toml
##Below is for centos/oracle. please check online for other distributions
sudo sed -i 's/^disabled_plugins = \["cri"\]/#&/' /etc/containerd/config.toml
systemctl restart containerd
systemctl status containerd

##Need to stop the swap, otherwise, kubelet will not run, Below is for Centos/oracle. please check online for other distributions
swapoff -a

## Install kubeadm,kubelet,kubectl
#sudo wget https://raw.githubusercontent.com/lerndevops/labs/master/scripts/installK8S.sh -P /tmp
sudo wget https://raw.githubusercontent.com/salwad-basha-shaik/labs/master/scripts/installK8S.sh -P /tmp
sudo chmod 755 /tmp/installK8S.sh
sudo bash /tmp/installK8S.sh
sudo systemctl start kubelet
sudo systemctl enable kubelet
sudo systemctl is-enabled kubelet
##wait for 15 seconds and validate below
##sleep 15
##sudo systemctl status kubelet

## Run Below on Master Node to get join token 

kubeadm token create --print-join-command 

    copy the kubeadm join token from master & run it on all worker nodes

    Ex: sudo kubeadm join 10.192.148.186:6443 --token or4owz.wfpu5ilsffgfn30q --discovery-token-ca-cert-hash sha256:4070621ea380c953d82d7ae05f49180ca3c3e68a0c74352ed6afd940ce75eb60 --cri-socket unix:///var/run/cri-dockerd.sock
```

## Install kubelet to access cluster by using kubeconfig file

step1:
```
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://pkgs.k8s.io/core:/stable:/v1.30/rpm/
enabled=1
gpgcheck=1
gpgkey=https://pkgs.k8s.io/core:/stable:/v1.30/rpm/repodata/repomd.xml.key
EOF
```

step2:
```
sudo yum install -y kubectl
```

step3:
```
kubectl version --client
```

step4: To access with kubeconfig
Setting Up with the Kubeconfig File

```
export KUBECONFIG=/path/to/your/kubeconfig.yaml

      ## above is only for temporary purpose but for global setup use below

echo 'export KUBECONFIG=/apps/rancher/amp-clm-sit-kubeconfig.yml' >> ~/.bashrc
source ~/.bashrc
```

step5: Check cluster information:
```
kubectl cluster-info
kubectl config get-contexts
kubectl get nodes
```

## shortcuts in linux alias 
```
vi ~/.bashrc
source ~/.bashrc
alias k='kubectl'
alias kgp='kubectl get pods -o wide'
alias kgd='kubectl get deploy -o wide'
alias kgs='kubectl get svc -o wide'
alias kga='kubectl get all'
alias context='echo kubectl config set-context --current --namespace='
alias restart-argocd='kubectl scale -n argocd deployment/argocd-server --replicas=0 && kubectl scale -n argocd deployment/argocd-server --replicas=1'
```

## Argocd CLI installation steps:
 
step1:
```
curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
 ```

step2:
```
sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd
```

step3:
```
argocd login sv2lxelaappr021:31110 --username admin --password <password>
```
 
step4: Adding cluster to argoCD
```
argocd cluster list
kubectl config get-contexts -o name
argocd cluster add kubernetes-admin@kubernetes
```

## To safely remove and re-add a worker node from your Kubernetes cluster, follow these steps:

step1:
```
kubectl cordon <node>
```

step2:
```
kubectl drain <node> --ignore-daemonsets --delete-emptydir-data
```

step3:
```
kubectl delete node <node>
```

Step 4: Reset the Node (on the Node Itself)
```
sudo kubeadm reset --cri-socket unix:///var/run/cri-dockerd.sock
sudo rm -rf /etc/cni/net.d /var/lib/cni/ /var/lib/kubelet /etc/kubernetes
```

step5: -- from master node
```
kubeadm token create --print-join-command  
```

step6:
```
kubeadm join 10.193.136.98:6443 --token fq0bmj.8cfppyw6t5vepmeu --discovery-token-ca-cert-hash sha256:acbff703277fc16e2b02bdb83aa85bd5be7bc9e5688f82e33728fbcc --cri-socket unix:///var/run/cri-dockerd.sock
```

# Kubernetes Dashboard - Standalone Version Setup

This repository provides instructions to deploy the Kubernetes Dashboard (v2.4.0) using an insecure setup for lab or development purposes.  
**Note:** This setup is **not recommended** for production use due to the lack of authentication.

## Prerequisites

- A running Kubernetes cluster
- `kubectl` configured to access the cluster

## Setup Instructions

### 1. Download the Dashboard YAML file

Download the standalone dashboard deployment manifest:

```
wget https://github.com/salvathshaik-orgg/labs/raw/refs/heads/master/kubernetes/dashboard/dashboard-insecure-v2.4.0.yml
```

### 2. Deploy the Dashboard

Apply the manifest using kubectl:

```
kubectl apply -f dashboard-insecure-v2.4.0.yml
```
⚠️ The dashboard will be deployed in the default namespace.

### 3. Accessing the Dashboard

Once deployed, you can access the dashboard using a node port

```
http://sv2lxampk8pr01:30009/#/pod?namespace=default
```

# 🐳 Docker Installation Issue - Repository Metadata Error

When attempting to run the following command on Oracle Linux:

```bash
sudo dnf -y install dnf-plugins-core
```

You may encounter the following error:

```
OEL7_ELS_BKP_BKP                                                                                       0.0  B/s |   0  B     00:00
Errors during downloading metadata for repository 'OEL7_ELS_BKP':
  - Curl error (37): Couldn't read a file:// file for file:///tmp/ExtendPatches/repodata/repomd.xml [Couldn't open file /tmp/ExtendPatches/repodata/repomd.xml]
Error: Failed to download metadata for repo 'OEL7_ELS_BKP': Cannot download repomd.xml: Cannot download repodata/repomd.xml: All mirrors were tried
```

---

## ✅ Solution

Disable the faulty repository and clear the DNF cache:

```bash
sudo dnf config-manager --disable OEL7_ELS_BKP
sudo dnf clean all
sudo rm -rf /var/cache/dnf
```

After cleaning up, **re-run the installation**:

```bash
sudo dnf -y install dnf-plugins-core
```

---

This step is part of the Docker installation process. If you encounter the metadata error, applying the above solution should allow you to proceed with your Docker setup.


# 📦 Kubernetes Yum Repo Error – `OEL7_ELS_BKP` and How to Fix It

When installing or upgrading Kubernetes components (e.g., `kubelet`, `kubeadm`, `kubectl`) on Oracle Linux 7 or 8, you may encounter the following error:

---

## ❗️ Error

```
Errors during downloading metadata for repository 'OEL7_ELS_BKP':
  - Curl error (37): Couldn't read a file:// file for file:///tmp/ExtendPatches/repodata/repomd.xml [Couldn't open file /tmp/ExtendPatches/repodata/repomd.xml]
Error: Failed to download metadata for repo 'OEL7_ELS_BKP': Cannot download repomd.xml: Cannot download repodata/repomd.xml: All mirrors were tried
```

---

## 📌 Root Cause

The system has a local (offline) YUM repository configured with the name `OEL7_ELS_BKP`. This repo points to a file location:

```
file:///tmp/ExtendPatches/
```

However, this file or path does **not exist**, causing `yum` to fail — even if the Kubernetes repo is correctly set up.

---

## ✅ Solutions

### Option 1: Disable the Broken Repo Temporarily

Use the `--disablerepo` flag while installing Kubernetes components:

```bash
sudo yum install -y kubelet-1.30.1 kubeadm-1.30.1 kubectl-1.30.1 --disablerepo=OEL7_ELS_BKP
```

---

### Option 2: Permanently Disable the Broken Repo

#### Method 1: Edit the `.repo` file manually

```bash
sudo vi /etc/yum.repos.d/OEL7_ELS_BKP.repo
```

Set the following:

```ini
enabled=0
```

#### Method 2: Use `yum-config-manager` (if installed)

```bash
sudo yum-config-manager --disable OEL7_ELS_BKP
```

---

## 📦 Retry the Kubernetes Installation

After disabling the broken repo:

```bash
sudo yum install -y kubelet-1.30.1 kubeadm-1.30.1 kubectl-1.30.1
```

---

## 📝 Notes

- This issue is unrelated to the Kubernetes repo itself — the error is caused by a broken local repo interfering with the entire YUM transaction.
- You can re-enable the repo later if you need it, or remove the `.repo` file if unused.

---

## 🔗 Related

- [Kubernetes Linux Install Docs](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)
- [Mirantis cri-dockerd](https://github.com/Mirantis/cri-dockerd)


# 🛠️ Kubernetes Worker Node Join Failure - TLS Bootstrap Timeout

## 📅 Date Recorded: 2025-07-14
## ✅ Status: Resolved

---

## 🔍 Issue Summary

While attempting to join a Kubernetes worker node to the cluster using `kubeadm join`, the process failed with the following error:

```
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap
error execution phase kubelet-start: context deadline exceeded
```

---

## 🧠 Root Cause

This issue was caused by **TLS bootstrap failing** due to either:
- Leftover old kubelet/kubeadm configs from before cert rotation.
- Misconfiguration of the kubelet.
- CRI socket misalignment (e.g. Docker with cri-dockerd).
- Mismatch between control plane and worker kubelet version.
- Residual metadata in `/var/lib/kubelet/` or `/etc/kubernetes/`.

> ⚠️ Kubernetes control plane certificates had been **rotated 1 month earlier**, which invalidated old TLS trust for kubelets trying to rejoin.

---

## ✅ Confirmed Conditions (no issues)

- ✅ Firewall was **inactive** on the worker node.
- ✅ SELinux was **disabled**.
- ✅ `curl -k https://<MASTER_IP>:6443/version` succeeded, confirming network and API reachability.

---

## 🧹 Solution Steps (Clean Fix Path)

### 🔁 Step 1: Cleanup the worker node

```bash
kubeadm reset -f
sudo systemctl stop kubelet
sudo systemctl stop docker

sudo rm -rf /etc/kubernetes/
sudo rm -rf /var/lib/kubelet/
sudo rm -rf /etc/cni/ /opt/cni/
sudo rm -rf ~/.kube
sudo rm -rf /etc/systemd/system/kubelet.service.d/

sudo systemctl daemon-reexec
sudo systemctl daemon-reload
```

---

### 🔁 Step 2: Ensure CRI socket is correct

```bash
ls -l /var/run/cri-dockerd.sock
```

> Must exist if using Docker runtime.
> Join command **must use**: `--cri-socket unix:///var/run/cri-dockerd.sock`

---

### 🔁 Step 3: Reinstall kubeadm, kubelet, kubectl (v1.30.1 to match control plane)

```bash
sudo yum install -y kubelet-1.30.1 kubeadm-1.30.1 kubectl-1.30.1
sudo systemctl enable kubelet
sudo systemctl restart kubelet
```

---

### 🔁 Step 4: Rejoin with new token

Generate on master:
```bash
kubeadm token create --print-join-command
```

Run the resulting command on worker node:

```bash
kubeadm join <MASTER_IP>:6443 \
  --token <token> \
  --discovery-token-ca-cert-hash sha256:<hash> \
  --cri-socket unix:///var/run/cri-dockerd.sock
```

---

## ✅ Outcome

The node successfully rejoined the cluster after following the above steps.

---

## 📝 Notes

- This issue is common after renewing Kubernetes CA or kubelet certs.
- Cleaning `/var/lib/kubelet` and `/etc/kubernetes` is critical.
- Always match Kubernetes versions between master and worker.


