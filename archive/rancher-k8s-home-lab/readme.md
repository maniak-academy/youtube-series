# Install Rancher k8s Lab at home in VMware Workstation

# Step 1 - Deploy 3 Virtual Machines (ubuntu)
- 2CPU
- 6144GiG
- 20HD

For your home lab, i suggest you set static IPs
sudo swapoff -a

sudo vim /etc/fstab

# Comment out lines that have the word swap on them

sudo reboot

# Step 2 - create the install script and deploy it 




# Step 3 Install Docker Rancher 

install rancher single node docker
https://docs.ranchermanager.rancher.io/pages-for-subheaders/rancher-on-a-single-node-with-docker

Install this on the first host iceman-01

Log into your host, and run the command below:

```
docker run -d --restart=unless-stopped \
  -p 80:80 -p 443:443 \
  --privileged \
  rancher/rancher:latest
```

# Step 4 install k8s 

Once docker has setup you will need to log in by getting the password

Find your container ID with docker ps, then run:
docker logs 6d0c9cff9dc1 2>&1 | grep "Bootstrap Password:"

# step 5 setup controler 

Setup a cluster called "iceman-00"

Change the network provider to Flannel 

We will remove the nginx ingress controller because we will do 

# Step 6 we will 

One iceman-01 we will install
- ETCd
- ControlPlan


Once this is finish we will deploy  "worker" on iceman-01 -03

- Show ranchger is making it happen

# Step 6 we will install 

# Step 7.. watch rancher build 

in the logs / 

# Step 8 install storage longhorn 

We need persistance storage to host consul


Click on apps ? type storage and install Longhorn

Longhorn is a lightweight, reliable and easy to use distributed block storage system for Kubernetes. Once deployed, users can leverage persistent volumes provided by Longhorn.

Longhorn creates a dedicated storage controller for each volume and synchronously replicates the volume across multiple replicas stored on multiple nodes. The storage controller and replicas are themselves orchestrated using Kubernetes. Longhorn supports snapshots, backups and even allows you to schedule recurring snapshots and backups!

Install loghorn into default

# Install Metallb

Go into apps

# connect to rancher in terminal / cli

- Download kubectl to your machine.
- https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/#install-using-native-package-management

- copy the kube config from the downloads. 


  mkdir -p $HOME/.kube
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

 chmod 0600  $HOME/.kube/config
 
# Install heml on machine
curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
sudo apt-get install apt-transport-https --yes
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm

# Configure Mettallb

Since we are doing this in our lab, we will just setup layer2 load balancing 

```
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: default
      protocol: layer2
      addresses:
      - 192.168.86.110-192.168.86.115
```


```
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: ippool
  namespace: metallb-system
spec:
  addresses:
  - 192.168.86.110-192.168.86.115
```


---
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: default
  namespace: metallb-system
spec:
  addresses:
  - 192.168.86.110-192.168.86.115
  autoAssign: true
  
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: default
  namespace: metallb-system
spec:
  ipAddressPools:
  - default