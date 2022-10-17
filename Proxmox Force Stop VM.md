### Releavant Links

- [[Creating Proxmox Image Templates]]

### Stopping a VM

- Attempt to stop VM via terminal using VM ID
```bash
qm stop 201
```

### Unlocking VM

- If error message returns VM is locked, try unlocking
```bash
qm unlock 201
```

### Fix Timeout Message

- If error message returns timeout, try killing process by first identifying the process by VM ID and then killing identified process. Note: I've had some issues with this method returning PID not found
```bash
ps aux | grep "/usr/bin/kvm -s 201" | awk '{print $6}' | xargs kill -9 $1
qm stop 201
```

- If all else fails reboot Proxmox machine
```bash
reboot
```