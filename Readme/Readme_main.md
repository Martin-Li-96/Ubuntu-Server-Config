# Ubuntu Server Config
## This clust will make up of one Mac and two ubuntu server (one has GPU, another don't have GPU)
### Installation of Ubuntu Server Edition
- HP server without GPU
  1. Following the Install Wizard; 
   (**Note: Choosing the network device **that can **connect** to the** internet if you have more than one network device**)
  2. Installing the OpenSSH-server while you install the Ubuntu;
  3. Network Config
       - Make sure you have **sudo** permission
       - Backup the original network config files
           ```bash
           $ sudo cp -r /etc/netplan/00-installer-config.yaml /etc/netplan/00-installer-config.yaml.bak
           #In Ubuntu Server 22.04 LTS the wifi config is a seperate config file, you also can set wifi and ethernet in one config file
           $ sudo cp -r /etc/netplan/00-installer-config-wifi.yaml /etc/netplan/00-installer-config-wifi.yaml
           ``` 
        - Set Static IP address for Ethernet (In this condition the HP server will use ethernet for the inner network and wifi for the internet)
           -  Editing the /etc/netplan/00-installer-config.yaml 
           ```bash
           $ sudo vi /etc/netplan/00-installer-config.yaml
           # Note you need to pay more attention to the structure of yaml file, especially the space
           ```
           After configuring the 00-installer-config.yaml is shown in the following figure.
           ![avatar](ethernet.png)
           Where addresses are the static IP addresses you will use, and use routes to set the gateway. The metric property is used to set the priority of the network device, the bigger number means low priority, and vice versa. In this condition, we set a bigger number of metric than the WiFi device.
       - Set Wi-Fi for internet
             ![avatar](wifi.png)   
         Changing wlx00e04c870000 to your wifi device name.
         ```bash
         access-points:
             [Your Wifi Name]:
                 password: XXXX
         ```
        - After all editing, Run:
            ```bash
            sudo netplan apply
            ```
  4. LVM extend storage (Optionanl) 
    
  5. Install & Configure k8s
      - Install docker [link](https://docs.docker.com/engine/install/debian/)
        ```bash
          sudo apt-get remove docker docker-engine docker.io containerd runc
          
          sudo apt-get update
          sudo apt-get install \ 
              ca-certificates \
              curl \
              gnupg \
              lsb-release

          sudo mkdir -p /etc/apt/keyrings
          curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

          echo \
          "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \
          $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

          sudo apt-get update

          sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin

          sudo systemctl enable docker
          sudo systemctl start docker
          sudo systemctl status docker       
        ```  
      - Install cri-dockerd  **(Run these commands as root)** [link](https://github.com/Mirantis/cri-dockerd)
        ```bash
          git clone https://github.com/Mirantis/cri-dockerd.git

          wget https://storage.googleapis.com/golang/getgo/installer_linux
          chmod +x ./installer_linux
          ./installer_linux
          source ~/.bash_profile

          cd cri-dockerd
          mkdir bin
          go build -o bin/cri-dockerd
          mkdir -p /usr/local/bin
          install -o root -g root -m 0755 bin/cri-dockerd /usr/local/bin/cri-dockerd
          cp -a packaging/systemd/* /etc/systemd/system
          sed -i -e 's,/usr/bin/cri-dockerd,/usr/local/bin/cri-dockerd,' /etc/systemd/system/cri-docker.service
          systemctl daemon-reload
          systemctl enable cri-docker.service
          systemctl enable --now cri-docker.socket

        ``` 
      - Install Kubernetes
        
        ```bash
        #Disable the SWAP partition
        sudo swapoff -a
        sudo rm /swap.img
        #edit /etc/fstab -> delete /swap.img none swap sw 0 0
    
        sudo apt-get update
        sudo apt-get install -y apt-transport-https ca-certificates curl
        sudo curl -fsSLo /etc/apt/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
        echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
        sudo apt-get update
        sudo apt-get install -y kubelet kubeadm kubectl
        sudo apt-mark hold kubelet kubeadm kubectl
        ```
     - Set Hostname
        ```bash
        #Master Node
        sudo hostnamectl set-hostname master-node
        #edit /etc/hosts
        ip_adress master-node
        #Work node
        sudo hostnamectl set-hostname w1
        ```
     - Disable the UFW for ubuntu's firewalld
        ```bash
        sudo systemctl stop ufw
        ``` 
     - Initialize Kubernetes on Master Node
       ```bash
       sudo kubeadm init --pod-network-cidr=192.168.2.2/24
       #Set your own IP addresses 

       #Run as root
       #vi .bashrch or /etc/profile
       export KUBECONFIG=/etc/kubernetes/admin.conf
       #please note if you edit /etc/profile you'd better add "source /etc/profile" to your .bashrc
       ```
     - Choosing Network for k8s
        https://kubernetes.io/docs/concepts/cluster-administration/addons/

        In this condition, we choose flannel

    - Add worker node to master node
      
      ```bash
      #Run on master node and copy the join info
      kubeadm token create --print-join-command
      #Run on worker node
      #past the join info
      ```
    - Option: edit label of worker node
      ```bash 
        kubectl label node nuc11-server node-role.kubernetes.io/work=worker 
      ```  
- NUC11 Server with gpu
  The config is the same as HP server, but don't exec kubectl init, but run add worker node  command

       

    
Reference: 

1. https://www.letscloud.io/community/how-to-install-kubernetesk8s-and-docker-on-ubuntu-2004
2. https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/