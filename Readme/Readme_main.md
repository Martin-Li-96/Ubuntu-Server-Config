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

  4. Install & Configure k8s
    

    

        

   


