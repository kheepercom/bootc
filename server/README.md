# server

This is the smallest server image we could design that boots on cloud instances and bare metal servers while also being suitable as a base for building your own images.
It's rebuilt nightly from Fedora 45's `minimal-plus` manifest, the shared base of Fedora IoT, Atomic Desktops and CoreOS.
We add chrony, iputils, and a default-deny firewall with nftables.
(Firewalld with its requirement for python was intentionally left out).

## Release

`us.kheeper.com/bootc/server:45` and `eu.kheeper.com/bootc/server:45`

## Contents

- kernel, systemd, bootc, bootupd, ostree, rpm-ostree, dnf5
- SELinux: `selinux-policy-targeted`, `container-selinux`
- Networking: NetworkManager, systemd-resolved, iproute, nftables, iputils
- SSH: openssh-server, openssh-clients
- Access: sudo, polkit
- Containers: podman, skopeo, crun, criu
- Storage: cryptsetup, lvm2, tar
- Hardware: linux-firmware, fwupd, tpm2-tools, zram-generator
- chrony, kmscon (console)

NetworkManager, sshd, chronyd and nftables are enabled.

## Derive

**You can't log in yet.** There are no users and root is locked. Derive from it to add a login and your application.

```Containerfile
FROM us.kheeper.com/bootc/server:45

RUN dnf install -y postgresql-server
```

Refer to [sibling directories](https://github.com/kheepercom/bootc/tree/main/headscale) for more examples.

## Firewall

`nftables.service` loads `/usr/lib/nftables/server.nft`: inbound is dropped except established traffic, loopback, ICMP, DHCPv6, podman bridges and SSH (22/tcp). Reloading or stopping the service only touches the `inet server` table, so podman's rules survive.

Open ports from a derived image with a drop-in:

```Containerfile
RUN echo 'add element inet server tcp_ports { 80, 443 }' > /usr/lib/nftables/server.d/myapp.nft
```

UDP ports go in `udp_ports`.

## Automatic updates

`bootc-fetch-apply-updates.timer` checks for a new image an hour after boot and every 8 hours after that (plus up to 2 hours of random delay), and reboots into it when it finds one.

## Need more?

This is not a general-purpose operating system.

[`quay.io/fedora/fedora-iot:45`](https://quay.io/repository/fedora/fedora-iot) is Fedora's official IoT bootc image, built on the same `minimal-plus` set. It adds firewalld, greenboot (health checks after an upgrade, with automatic rollback), ignition (first-boot provisioning), clevis with TPM2 (automatic LUKS unlock), FDO onboarding, usbguard, and Wi-Fi and WWAN support.

[`quay.io/fedora/fedora-bootc:45`](https://quay.io/repository/fedora/fedora-bootc) is the standard image: a general-purpose base, roughly a headless server install, with automatic updates on.
