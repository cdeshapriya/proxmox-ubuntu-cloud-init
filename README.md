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

  507  sudo virt-customize -a jammy-server-cloudimg-amd64.img --root-password file:password_root.txt
  508  sudo virt-customize -a jammy-server-cloudimg-amd64.img --run-command "useradd -m -s /bin/bash ubuntu"
  509  sudo virt-customize -a jammy-server-cloudimg-amd64.img --password myuser:file:password_ubuntu.txt
  510  sudo virt-customize -a jammy-server-cloudimg-amd64.img --run-command "apt-add-repository ppa:fish-shell/release-3 --yes"
  511  sudo virt-customize -a jammy-server-cloudimg-amd64.img --install fish
  512  sudo virt-customize -a jammy-server-cloudimg-amd64.img --run-command "chsh -s /usr/bin/fish ubuntu"
  513  sudo virt-customize -a jammy-server-cloudimg-amd64.img --update
  514  sudo qm create 1001 --name "ubuntu-22.04-template" --memory 8192 --cores 8 --net0 virtio,bridge=vmbr0
  515  sudo qm importdisk 1001 jammy-server-cloudimg-amd64.img vdata
  516  sudo qm set 1001 --scsihw virtio-scsi-pci --scsi0 vdata:vm-1001-disk-0
  517  sudo qm set 1001 --boot c --bootdisk scsi0
  518  sudo qm set 1001 --ide2 vdata:cloudinit
  519  sudo qm set 1001 --serial0 socket --vga serial0
  520  sudo qm set 1001 --agent enabled=1
  521  sudo qm template 1001
  522  history > create-vm-tmpl.txt
