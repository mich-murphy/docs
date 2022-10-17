>Refer to [technotim blog](https://docs.technotim.live/posts/cloud-init-cloud-image/)

### Find & Configure Image

- SSH into Proxmox machine
```bash
ssh user@ip
```

- Find download url for cloud image distribution (pre-configured images meant for cloud deployment) and download
```bash
wget https://cloud-images.ubuntu.com/focal/current/focal-server-cloudimg-amd64.img
```

- Create new virtual machine, specifying the image ID, name and available RAM and CPU cores
```bash
qm create 100 --memory 6144 --core 6 --name ubuntu-20.04 --net0 virtio,bridge=vmbr0
```

- Import downloaded image to local lvm storage
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

### Setup Image Template

**DO NOT START YOUR VM**

- Configure hardware and cloud init (done inside of Proxmox in web browser). I prefer to configure HDD size after template has been cloned, depending on use case
- Create template
```bash
qm template 100
```

- Clone template - template ID (100), new image ID (201)
```bash
qm clone 100 201 --name media --full
```