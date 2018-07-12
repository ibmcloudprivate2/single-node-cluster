# single-node-cluster

new environment

```
$ vagrant init
```

[Prepare your cluster for installation](https://www.ibm.com/support/knowledgecenter/en/SSBS6K_2.1.0.3/installing/prep_cluster.html)

To the file provided

add guest addition

1. Mount the VirtualBox Guest Additions ISO:
```
mount /dev/cdrom /mnt
```
2. Run the installer:
```
sh /mnt/VBoxLinuxAdditions.run
```
3. Unmount the ISO
```
umount /mnt
```
4. Restart the guest...

Install socat

1. [Download socat](https://centos.pkgs.org/7/repoforge-x86_64/socat-1.7.2.4-1.el7.rf.x86_64.rpm.html)

place the file in the same folder of Vagrantfile

2. start and login into CentOS
```
$ vagrant up
$ vagrant ssh
```

3. go to shared folder of host OS
```
$ cd /vagrant
```

4. Install socat
```
$ sudo yum install socat-1.7.2.4-1.el7.rf.x86_64.rpm
```

5. ensure python 2.7.x is installed
```
$ python --versionh
```

Install ICP 2.1.0.3
```
mkdir /opt/ibm-cloud-private-2.1.0.3  
cd /opt/ibm-cloud-private-2.1.0.3
```

install [docker](https://www.ibm.com/support/knowledgecenter/en/SSBS6K_2.1.0.3/installing/install_docker.html#docker_icp)
```
chmod +x icp-docker-17.12.1_x86_64.bin
sudo ./icp-docker-17.12.1_x86_64.bin --install
```

download [docker ce](https://download.docker.com/linux/centos/7/x86_64/stable/Packages/)

```
 sudo yum install  docker-ce-17.12.1.ce-1.el7.centos.x86_64.rpm
```

start docker engine
```
sudo systemctl start docker
sudo groupadd docker
sudo usermod -aG docker $USER
sudo systemctl enable docker
```

Extract the images and load them into Docker. Extracting the images might take a few minutes.

install docker
```
sudo chmod +x icp-docker-17.09_x86_64.bin
sudo ./icp-docker-17.09_x86_64.bin –install
```

```
tar xf ibm-cloud-private-x86_64-2.1.0.3.tar.gz -O | sudo docker load
```

Create an installation directory
```
mkdir /opt/ibm-cloud-private-2.1.0.3
cd /opt/ibm-cloud-private-2.1.0.3
```

Extract the configuration files from the installer image.
```
sudo docker run -v $(pwd):/data -e LICENSE=accept ibmcom/icp-inception:2.1.0.3-ee cp -r cluster /data
```

generate key in boot node
```
ssh-keygen -b 4096 -f ~/.ssh/id_rsa -N ""
```

Add the key to the list of authorized keys.
```
cat ~/.ssh/id_rsa.pub | sudo tee -a ~/.ssh/authorized_keys
```

```
sudo cp ~/.ssh/id_rsa ./cluster/ssh_key
ssh-copy-id -i ~/.ssh/id_rsa.pub vagrant@192.168.34.10
```

[ICP offline installation resource](https://medium.com/ibm-cloud/ibm-cloud-private-offline-installation-eb730ae13bfc)

deploy install
```
cd ./cluster
sudo docker run --net=host -t -e LICENSE=accept -v "$(pwd)":/installer/cluster ibmcom/icp-inception:2.1.0.3-ee install
```

install kubectl
```
sudo docker run -e LICENSE=accept --net=host -v /usr/local/bin:/data ibmcom/icp-inception:2.1.0.3-ee cp /usr/local/bin/kubectl /data
```

install helm
```
docker run -e LICENSE=accept --net=host -v /usr/local/bin:/data ibmcom/icp-helm-api:1.0.0 cp /usr/src/app/public/cli/linux-amd64/helm /data
```

Download [bluxmix cli](https://console.bluemix.net/docs/cli/reference/bluemix_cli/download_cli.html#install_use)
install [ICP CLI](https://www.ibm.com/support/knowledgecenter/SSBS6K_2.1.0.3/manage_cluster/install_cli.html)

```
tar xvf IBM_Cloud_CLI_0.7.1_amd64.tar.gz
./Bluemix_CLI/install_bluemix_cli
```

To install the IBM Cloud Private CLI,
```
bx plugin install ./icp-linux-amd64
```

login to cluster
```
bx pr login -a https://192.168.34.10:8443 --skip-ssl-validation
sudo docker login mycluster.icp:8500
```

Install the file from Passport Advantage
```
bx pr load-ppa-archive --archive <compressed_file_name> [--clustername <cluster_CA_domain>] [--namespace <namespace>]
```

```
tar xf ICP-EE-2.1.0.3.tar.gz -O | docker load
```

Load MQ image into ICP
```
bx pr load-ppa-archive --archive IBM_MQ_9.0.5.0_LINx8664_ICP_2.1.0.tar.gz
```

Load CAM image into ICP
```
bx pr load-ppa-archive --archive icp-cam-x86_64-2.1.0.3_06-27.tar.gz
```

Running Docker API commands
```
export CMD=`sudo curl --cacert /etc/docker/certs.d/mycluster.icp:8500/ca.crt -s -u admin:admin "https://192.168.34.10:8443/image-manager/api/v1/auth/token?service=token-service&scope=registry:catalog:*"`
export ID_TOKEN=$(echo $CMD | python -c 'import sys,json; print json.load(sys.stdin)["token"]')
echo $ID_TOKEN

curl --cacert /etc/docker/certs.d/mycluster.icp:8500/ca.crt -s -H "Authorization: Bearer $ID_TOKEN" "https://<cluster_CA_domain>:8500/v2/_catalog"
```

uninstall ICP
```
docker run -e LICENSE=accept --net=host -t -v "$(pwd)":/installer/cluster ibmcom/icp-inception:2.1.0.3-ee uninstall
service docker restart
```

```
/etc/ssh/sshd_config
systemctl disable firewalld
systemctl stop firewalld
systemctl status firewalld
```

```
mkdir -p cluster/images
sudo mv /vagrant/ibm-cloud-private-x86_64-2.1.0.3.tar.gz  cluster/images/
```

# resize volume in centos

```
sudo fdisk -l
fdisk /dev/sdb
sudo pvcreate /dev/sdb1
sudo vgdisplay
sudo vgextend VolGroup00 /dev/sdb1
sudo lvextend -l +100%FREE /dev/mapper/VolGroup00-LogVol00
sudo xfs_growfs /dev/mapper/VolGroup00-LogVol00
```

# resize virtualbox instance disk size

```
vboxmanage clonehd "/Users/jaricsng/VirtualBox VMs/local.single.icp/centos-7-1-1.x86_64.vmdk" "/Users/jaricsng/VirtualBox VMs/local.single.icp/centos-7-1-1.x86_64.vdi" --format VDI
```

```
vboxmanage modifyhd "/Users/jaricsng/VirtualBox VMs/local.single.icp/centos-7-1-1.x86_64.vdi" --resize 102400
```
