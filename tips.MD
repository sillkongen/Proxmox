### Find next free VM ID
```
pvesh get /cluster/nextid
```

### Create containter with external and internal nets
```
pct create 100 \
    local:vztmpl/ubuntu-16.04-standard_16.04-1_amd64.tar.gz \
    --cores 2 --cpuunits 1024 \
    --memory 4096 --swap 128 \
    --hostname ct100.example.com \
    --net0 name=eth0,ip=1.2.3.4/32,bridge=vmbr0,gw=1.2.3.0 \
    --net1 name=eth1,ip=192.168.0.2/32,bridge=vmbr1 \
    --rootfs local:24 \
    --onboot 1
  ```
  
### Create container with pvesh command
```
pvesh create /nodes/promox01/lxc \
      -vmid 1001 \
      -ostemplate local:vztmpl/ubuntu-16.04-x64-base.tar.gz \
      -cores 2 -cpuunits 1024 \
      -memory 4096 -swap 128 \
      -hostname ct1001.example.com \
      -net1 name=eth1,ip=192.168.0.2/32,bridge=vmbr1 \
      -rootfs local:24 \
      -onboot 1
```

### Create KVM with pvesh command
```
pvesh create /nodes/kvm01/storage/local/content \
     -filename vm-101-disk-0.qcow2 \
     -format qcow2 -size 32G -vmid 101

pvesh create /nodes/kvm01/qemu \
      -vmid 101 -memory 2048 \
      -sockets 1 -cores 4 \
      -net0 e1000,bridge=vmbr0 \
      -net1 e1000,bridge=vmbr1 \
      -ide0=local:101/vm-101-disk-0.qcow2 \
      -ide2 local:iso/ubuntu-14.04-server-amd64.iso,media=cdrom
      
pvesh create /nodes/kvm01/qemu/101/status/start
```
  
### Remove container. Be careful, it'll destroy ct without questions
```
pct destroy 100
```

### View sum of memory allocated to VMs and CTs
```
grep -R memory /etc/pve/local | awk '{sum += $NF } END {print sum;}'
```

### View sorted list of vmid
```
cat /etc/pve/.vmlist | grep node | cut -d '"' -f2 | sort -n
```

### View sorted list of VMs like `vmid proxmox_host type`
```
cat /etc/pve/.vmlist | grep node | tr -d '":,'| awk '{print $1" "$4" "$6 }' | sort -n | column -t
```

### View sorted list of VMs like `vmid proxmox_host type vm_name`
```
for i in $(cat /etc/pve/.vmlist | grep node | cut -d '"' -f2 | sort -n);do NAME=$(grep -R 'name:' /etc/pve/nodes/*/*/$i.conf | awk {'print $2'}); INFO=$(grep $i /etc/pve/.vmlist | grep node | tr -d '":,'| awk '{print $1"\t"$4"\t"$6 }'); printf "%s\t%s\n" "$INFO" "$NAME" ;done
```
