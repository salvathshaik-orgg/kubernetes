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

## Install kubeadm,kubelet,kubectl
#sudo wget https://raw.githubusercontent.com/lerndevops/labs/master/scripts/installK8S.sh -P /tmp
sudo wget https://raw.githubusercontent.com/salwad-basha-shaik/labs/master/scripts/installK8S.sh -P /tmp
sudo chmod 755 /tmp/installK8S.sh
sudo bash /tmp/installK8S.sh
#wait for 15 seconds and validate below
##sleep 15
##sudo systemctl status kubelet

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
