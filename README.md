# free5gc-s3-docker
create free5gc-stange3-dockerfile and build docker image to deploy
参考free5gc,以stage3进行部署

主要步骤：
1、准备ubuntu 18/04,更换内核到5.0.0-23-generic
  apt-get update
  apt-get install linux-image-5.0.0-23-generic
  apt-get install linux-headers-5.0.0-23-generic
  dpkg --get-selections |grep linux-image  //检查安装的内核
  sudo apt-get remove //移除多余内核
  update-grub  //更新启动项
  reboot

2、安装docker
sudo apt-get -y install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo apt-key fingerprint 0EBFCD88
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) \
  stable"
sudo apt-get update
sudo apt-get -y install docker-ce docker-ce-cli containerd.io

3、安装k8s
  curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
  cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
  deb https://apt.kubernetes.io/ kubernetes-xenial main
   EOF
   sudo apt-get update
   apt-get install -y kubelet kubeadm kubectl

   kubeadm init --apiserver-advertise-address=192.168.0.24  --pod-network-cidr=10.244.0.0/16
