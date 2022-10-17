### Relavant Links

- [[Proxmox Force Stop VM]]
- [Technotim Blog](https://docs.technotim.live/posts/cloud-init-cloud-image/)
- [Austins Nerdy Things Blog](https://austinsnerdythings.com/2021/08/30/how-to-create-a-proxmox-ubuntu-cloud-init-image/)

### Find & Configure Image

- SSH into Proxmox machine (root@proxmoxip)
```bash
ssh root@192.168.1.5
```

- Find download url for cloud image distribution (pre-configured images meant for cloud deployment - use a KVM image for Proxmox) and download
```bash
wget https://cloud-images.ubuntu.com/focal/current/focal-server-cloudimg-amd64.img
```

- **The following is only necessary for an Ubuntu image**
	- The Ubuntu cloud image doesn't come with QEMU installed, we can add this to the image by first installing virt-customize
	```bash
	apt install libguestfs-tools -y
	```
	
	- Install QEMU Agent
	```bash
	virt-customize -a focal-server-cloudimg-amd64.img --install qemu-guest-agent
	```

	- Set password for root user
	```bash
	virt-customize -a bionic-server-cloudimg-amd64.img --root-password password:<pass>
	```

### Add Image to Proxmox

- Create new virtual machine, specifying the image ID, name and available RAM and CPU cores
```bash
qm create 100 --memory 2048 --cores 2 --name ubuntu-20.04 --net0 virtio,bridge=vmbr0
```

- Import downloaded image to local lvm storage (if not using lvm storage you can specify alternative e.g. local-zfs)
```bash
qm importdisk 100 focal-server-cloudimg-amd64.img local-lvm
```

- Attach new disk to vm
```bash
qm set 100 --scsihw virtio-scsi-pci --scsi0 local-lvm:vm-100-disk-0
```

- Add cloud init drive
```bash
qm set 100 --ide2 local-lvm:cloudinit
```

- Make cloud init drive bootable and restrict BIOS to boot only from disk
```bash
qm set 100 --boot c --bootdisk scsi0
```

- Add serial console
```bash
qm set 100 --serial0 socket --vga serial0
```

- Turn on QEMU Agent (allows for IP address visibility and VM control features)
```bash
qm set 100 --agent enabled=1
```

### Setup Image Template

- Setup cloud-init user/login and SSH key via Proxmox web interface
- Create template
```bash
qm template 100
```

- Clone template - template ID (100), new image ID (201), `--full` removes any connection of the new image to the template from which it was cloned
```bash
qm clone 100 201 --name media --full
```

- Setup IP address (note these setting are applied to the VM ID we created above 201)
```bash
qm set 201 --ipconfig0 ip=192.168.1.10/24,gw=192.168.1.1
```

### Starting & Connecting to VM

- Start VM
```bash
qm start 201
```

- Login via SSH (default user is ubuntu for Ubuntu Server)
```bash
ssh ubuntu@192.168.1.10
```

### Shutdown & Remove Image

- Shutdown and destroy VM
```bash
qm stop 201 && qm destroy 201
```