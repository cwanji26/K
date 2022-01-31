
Step 1: Configure Docker
apt-get update
apt-get -y install apt-transport-https ca-certificates curl gnupg-agent software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"apt-get update && apt-get -y install docker-ce docker-ce-cli
systemctl start docker

Step 2: Kernel Parameter Configuration
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1

EOF
sudo sysctl --system
Step 3:Installation (Master+Worker)
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF

apt-get update
apt-cache madison kubeadm
apt-get install -qy kubeadm=1.19.4.-00 kubelet=1.19.4-00 kubectl=1.19.4-00
apt-mark hold kubelet kubeadm kubectl
kubeadm version
kubeadm init --pod-network-cidr=10.244.0.0/16
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

Upgrade Steps
apt update
apt-cache madison kubeadm
apt-mark hold kubeadm
apt-mark unhold kubeadm &&
apt-get update && apt-get install -y kubeadm=1.20.4-00 &&
kubeadm version
kubeadm upgrade plan
kubeadm upgrade apply v1.20.4

replace x with the patch version you picked for this upgrade
SUCCESS! Your cluster was upgraded to "v1.20.x". Enjoy!
kubectl drain --ignore-daemonsets
apt-mark unhold kubelet kubectl &&
apt-get update && apt-get install -y kubelet=1.20.4-00 kubectl=1.20.4-00 &&
apt-mark hold kubelet kubectl
systemctl daemon-reload
systemctl restart kubelet
kubectl uncordon
