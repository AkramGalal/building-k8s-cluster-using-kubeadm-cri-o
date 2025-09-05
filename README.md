# Building K8s Cluster using Kubeadm and CRI-O container Runtime
This project demonstrates how to manually set up a Kubernetes (K8s) cluster from scratch using **Kubeadm** and **CRI-O** container runtime. The cluster configuration consists of 3 control-plane (master) nodes for high availability and 2 worker nodes for running applications.

## Setup and Deployment
### 1. Create Vagrantfile
- Install VirtualBox.
- Install Vagrant.
- Clone the repository.
- Use the Vagrantfile to define 5 VMs: 3 masters and 2 workers. Each VM has 2 CPU, 2 GB RAM, 20 GB HDD with image Linux Ubuntu 22.04.
- Install Vagrant plugins.
  ```bash
  vagrant plugin install vagrant-hostmanager
- Start creating VMs
  ```bash
  vagrant up

### 2. Update System and Install Required Packages
- Alternatively, run "script.sh" on all nodes to complete steps 2 to 4.
  
  ```bash
  sudo -i
  apt update
  sudo apt install -y apt-transport-https ca-certificates curl gnupg2 software-properties-common

### 3. Install and Configure CRI-O Container Runtime
- Add CRI-O Repository
  ```bash
  export OS=xUbuntu_22.04
  export CRIO_VERSION=1.24
  
  echo "deb https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/ /" | \
      sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list
    
  echo "deb http://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable:/cri-o:/$CRIO_VERSION/$OS/ /" | \
      sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable:cri-o:$CRIO_VERSION.list

- Import GPG Keys for the CRI-O repository
  ```bash
  curl -L https://download.opensuse.org/repositories/devel:kubic:libcontainers:stable:cri-o:$CRIO_VERSION/$OS/Release.key | sudo apt-key add -
  curl -L https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/Release.key | sudo apt-key add -

- Install CRI-O
  ```bash
  sudo apt update -y
  sudo apt install -y cri-o cri-o-runc

- Install CNI plugins for CRI-O
  ```bash
  sudo apt install -y containernetworking-plugins

- Configure CRI-O
  ```bash
  cat <<EOF | sudo tee /etc/crio/crio.conf
  [crio.runtime]
  root = "/var/lib/crio"
  
  [crio.network]
  network_dir = "/etc/cni/net.d/"
  plugin_dirs = [
      "/usr/lib/cni",
      "/opt/cni/bin"
  ]
  EOF
  
  sudo systemctl enable --now crio
  
- Install CRI-O tools and enable auto completion
  ```bash
  sudo apt install -y cri-tools
  sudo crictl --runtime-endpoint unix:///var/run/crio/crio.sock version
  sudo crictl completion > /etc/bash_completion.d/crictl
  source ~/.bashrc

### 4. Install Kubeadm, Kubelet, and Kubectl
- Open Required Ports
  ```bash
  sudo iptables -A INPUT -p tcp --dport 6443 -j ACCEPT

- Disable Swap
  ```bash
  sudo swapoff -a
  sudo sed -i '/swap/s/^/#/' /etc/fstab

- Add Kubernetes Repository
  ```bash
  sudo apt-get install -y apt-transport-https ca-certificates curl gpg
  sudo mkdir -p -m 755 /etc/apt/keyrings
  curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.32/deb/Release.key | \
      sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
  echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.32/deb/ /' | \
      sudo tee /etc/apt/sources.list.d/kubernetes.list
  sudo apt-get update

- Install Kubernetes components
  ```bash
  sudo apt-get install -y kubelet kubeadm kubectl
  sudo apt-mark hold kubelet kubeadm kubectl
  sudo systemctl enable --now kubelet

- Enable IP forwarding
  ```bash
  sudo sed -i '/^net.ipv4.ip_forward=/{h;s/=.*/=1/};${x;/^$/{s//net.ipv4.ip_forward=1/;H};x}' /etc/sysctl.conf
  sudo sysctl -p

### 5. Initiate Kubeadm control plane configuration on the master node.
- Best practice: Use a load balancer IP for high availability. In this project the IP of master1 node will be used as the primary master node for the cluster.
  ```bash
  sudo kubeadm init --control-plane-endpoint "10.0.0.10:6443"
- eplace <10.0.0.10> with the actual IP of master1 node.

- Set up K8s configuration for the current user
  ```bash
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

### 6. Join Worker Nodes
- Use the join command generated from step 6 on each worker node in the cluster
  ```bash
  sudo kubeadm join 10.0.0.10:6443 --token 56zoor.0owcsp9qtb2shsf3 \ --discovery-token-ca-cert-hash sha256:23c1b5bab2e09d06c86ac1935f7c0922bbf129eedb109e6ee3bb585d19a8ee3c
- Replace <10.0.0.10> with the actual IP of master1 node.

### 7. Add Additional Master Nodes for High Availability  
- Generate certification key.
  ```bash
    sudo kubeadm init phase upload-certs --upload-certs
- Use the generated certification to generate the join command.
  ```bash
  sudo kubeadm token create --print-join-command --certificate-key <CERT_KEY>
- This will generate a join command, run this generated join command on master2 and master3 to add them to the cluster.

### 8. Install Calico Network Plugin (CNI)
- Calico manages networking between pods and services, ensuring proper pod-to-pod communication.
  ```bash
  kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/calico.yaml

### 9. Validate Cluster  
- Verify that all master and worker nodes are in Ready state.
  ```bash
  kubectl get nodes -o wide
