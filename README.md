# Create an Ubuntu 22.04 template for Proxmox utilizing the Ubuntu 22.04 cloud-init image

## Set up a directory to download and store Ubuntu cloud images. 

```
mkdir -p cloud/images/ubuntu
```
## Change to the directory created

```
cd cloud/images/ubuntu/
```

## Install libguestfs tools for accessing and modifying virtual machine disk images

```
sudo apt update -y && sudo apt install libguestfs-tools -y
```

## Download Ubuntu  Cloud Image 

```
wget https://cloud-images.ubuntu.com/jammy/current/jammy-server-cloudimg-amd64.img
```

# Install qemu-guest-agent on the image. 

Additional packages can be specified by separating with a comma.

```
virt-customize -a jammy-server-cloudimg-amd64.img --install qemu-guest-agent, net-tools, git
```

## Setup root password file

```
ROOTPASS=$(LC_ALL=C </dev/urandom tr -dc A-Za-z0-9 2> /dev/null | head -c 28
```
```
echo $ROOTPASS > password_root.txt
```

## Setup ubunu admin user password file

```
UBUNTUPASS=$(LC_ALL=C </dev/urandom tr -dc A-Za-z0-9 2> /dev/null | head -c 28
```
```
echo $UBUNTUPASS > password_ubuntu.txt
```

## Setup root user  password

```
sudo virt-customize -a jammy-server-cloudimg-amd64.img --root-password file:password_root.txt
```

## Add ubuntu user as an admin user

```
sudo virt-customize -a jammy-server-cloudimg-amd64.img --run-command "useradd -m -s /bin/bash ubuntu"
```

## Add ubuntu user into sudo group

```
sudo virt-customize -a jammy-server-cloudimg-amd64.img --run-command "usermod -a -G sudo ubuntu"
```

## Set ubuntu user password

```
sudo virt-customize -a jammy-server-cloudimg-amd64.img --password ubuntu:file:password_ubuntu.txt
```

## Add additional fish shell repository

```
sudo virt-customize -a jammy-server-cloudimg-amd64.img --run-command "apt-add-repository ppa:fish-shell/release-3 --yes"
```

## Install fish shell

```
sudo virt-customize -a jammy-server-cloudimg-amd64.img --install fish
```

## Setup fish shell to ubuntu user

```
sudo virt-customize -a jammy-server-cloudimg-amd64.img --run-command "chsh -s /usr/bin/fish ubuntu"
```

## Update the image 

```
sudo virt-customize -a jammy-server-cloudimg-amd64.img --update
```

## Create the vm

```
  514  sudo qm create 1001 --name "ubuntu-22.04-template" --memory 8192 --cores 8 --net0 virtio,bridge=vmbr0
```

## Import the image as the vm disk

```
sudo qm importdisk 1001 jammy-server-cloudimg-amd64.img vdata
```

Note: Change the ```vdata``` as per your storage name

  516  sudo qm set 1001 --scsihw virtio-scsi-pci --scsi0 vdata:vm-1001-disk-0
  517  sudo qm set 1001 --boot c --bootdisk scsi0
  518  sudo qm set 1001 --ide2 vdata:cloudinit
  519  sudo qm set 1001 --serial0 socket --vga serial0
  520  sudo qm set 1001 --agent enabled=1
  521  sudo qm template 1001
  522  history > create-vm-tmpl.txt
