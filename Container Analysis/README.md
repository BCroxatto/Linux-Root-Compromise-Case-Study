**systemd-nspawn Container Architecture**

The directory /var/lib/machines contained multiple containers:
- router
- noted
- saas
- wat
- hostcontainer

Each had -upper and -work directories, indicating overlayfs-backed containers.

**systemd Service Configuration**

Containers were launched using:
- /lib/systemd/system/systemd-nspawn@.service

Enabled via:
- /etc/systemd/system/machines.target.wants/

Relevant configuration excerpts:

- Delegate=yes
- DevicePolicy=closed
- DeviceAllow=/dev/net/tun rmw
- DeviceAllow=/dev/fuse rwm
- DeviceAllow=/dev/loop-control rw
- DeviceAllow=block-loop rw
- DeviceAllow=/dev/mapper/control rw
- DeviceAllow=block-device-mapper rw

**Security Impact**
- Containers start as root
- Access to loop devices granted
- Access to device-mapper granted
- Delegate=yes allows extended resource control
- Containers can interact with host-critical subsystems

This configuration breaks expected container isolation guarantees.

**SSH Trust Material**

Authorized keys were discovered in:

Host:
- /root/.ssh/authorized_keys

Containers:
- /var/lib/machines/router-upper/root/.ssh/authorized_keys
- /var/lib/machines/hostcontainer-upper/root/.ssh/authorized_keys

Implications:

- Root trust material exists inside writable container filesystems
- Key extraction or modification could lead to persistent host access
- Host/container trust boundaries are blurred
