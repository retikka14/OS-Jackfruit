# Multi-Container Runtime

## Team

| Name     | SRN  |
| -------- | ---- |
| Member 1 | SRN1 |
| Member 2 | SRN2 |

## Build & Run

```bash
sudo apt update
sudo apt install -y build-essential linux-headers-$(uname -r) make gcc wget

mkdir rootfs-base
wget https://dl-cdn.alpinelinux.org/alpine/v3.20/releases/x86_64/alpine-minirootfs-3.20.3-x86_64.tar.gz
tar -xzf alpine-minirootfs-3.20.3-x86_64.tar.gz -C rootfs-base

make
sudo insmod monitor.ko
ls -l /dev/container_monitor

sudo ./engine supervisor ./rootfs-base

cp -a rootfs-base rootfs-alpha
cp -a rootfs-base rootfs-beta

sudo ./engine start alpha ./rootfs-alpha /bin/sh --soft-mib 48 --hard-mib 80
sudo ./engine start beta  ./rootfs-beta  /bin/sh --soft-mib 64 --hard-mib 96

sudo ./engine ps
sudo ./engine logs alpha

sudo ./engine stop alpha
sudo ./engine stop beta

dmesg | tail
sudo rmmod monitor
make clean
```

## Demo

* Multi-container: 2 containers under one supervisor
* Metadata: `engine ps`
* Logging: logs + buffer activity
* CLI/IPC: command + response
* Soft limit: warning in `dmesg`
* Hard limit: kill + metadata update
* Scheduling: workload comparison
* Cleanup: no zombies

## Analysis

Isolation: namespaces + chroot, kernel shared
Supervisor: manages lifecycle, signals, reaping
IPC: pipes (logs), socket (CLI), mutex+cond for buffer
Memory: RSS tracked, soft warn, hard kill (kernel enforced)
Scheduling: CFS, nice affects CPU share

## Design

Namespaces: isolation vs complexity
Supervisor: control vs single failure
Buffer: safe logs vs blocking
Kernel module: accuracy vs risk
Experiments: simple vs less precise

## Results

| Workload  | Result |
| --------- | ------ |
| High nice | slower |
| Low nice  | faster |

CFS gives fairness, not equal runtime
