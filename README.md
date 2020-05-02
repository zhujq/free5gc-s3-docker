# free5gc-s3-docker
create free5gc-stange3-dockerfile and build docker image to deploy
参考free5gc,以stage3进行部署,以k8s作为vim，mano、nfvo、除amf/upf外用k8s部署，amf、upf直接以可执行程序部署在ubuntu18.04上，规避iptables对sctp的支持不足，以及提高UPF处理能力。

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

sudo apt-get -y install apt-transport-https ca-certificates curl gnupg-agent software-properties-common

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

sudo apt-key fingerprint 0EBFCD88

sudo add-apt-repository  "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

sudo apt-get update

sudo apt-get -y install docker-ce docker-ce-cli containerd.io

3、安装k8s

  curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
  
  cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
  
  deb https://apt.kubernetes.io/ kubernetes-xenial main
   
  EOF
  
  sudo apt-get update
  
  apt-get install -y kubelet kubeadm kubectl
  
  kubeadm init --apiserver-advertise-address=xx.xx.xx.xx  --pod-network-cidr=10.244.0.0/16
   
  mkdir -p $HOME/.kube
   
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
  
  kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
  
  4、安装openvswitch和ovs-cni容器多端口插件
  
    apt install openvswitch-switch -y
    
    ovs-vsctl add-br br1
    
    kubectl apply -f ovs-cni.yaml
    
    kubectl apply -f ovs-net-crd.yaml
    
    //br1的IP地址设置为192.168.3.254/23，作为k8s 5gc网元的网口的网关
    
    vi /etc/netplan/50-cloud-init.yaml 
    
    sudo  netplan apply 
    
   5、安装 etcd-operator 和 Node Exporter
   
      cd etcd-cluster/rbac/
      
      ./create_role.sh
      
      cd ..
      
      kubectl apply -f ./
      
      cd ..
      
      kubectl apply -f prom-node-exporter.yaml
   
   6、部署mysql、mano、nfvo
   
     kubectl apply -f service-account-agent.yaml
     
     kubectl apply -f mysql-agent.yaml
     
     kubectl apply -f kube5gnfvo.yaml
     
     kubectl apply -f 5gmano-deploy.ymal
    
    7、部署除AFM/UPF外的5GC
    
     kubectl apply -f unix-daemonset.yaml
     
     kubectl apply -f free5gc-mongodb.yaml
     
     kubectl apply -f free5gc-configmap.yaml
     
     kubectl apply -f free5gc-nrf.yaml
      
