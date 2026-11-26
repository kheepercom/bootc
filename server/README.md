# server

This is the smallest server image we could design that boots on cloud instances and bare metal servers while also being suitable as a base for building your own images.
It's rebuilt nightly from Fedora 45's `minimal-plus` manifest, the shared base of Fedora IoT, Atomic Desktops and CoreOS, minus rpm-ostree, zram-generator, fwupd, kmscon, iptables-nft and criu (see `server.yaml`).
We add chrony, iputils, and a default-deny firewall with nftables.

## Contents

`us.kheeper.com/bootc/server:45` and `eu.kheeper.com/bootc/server:45`

- kernel, systemd, bootc, bootupd, ostree, dnf5
- SELinux: `selinux-policy-targeted`, `container-selinux`
- Networking: NetworkManager, systemd-resolved, iproute, nftables, iputils
- SSH: openssh-server, openssh-clients
- Access: sudo, polkit
- Containers: podman, skopeo, crun
- Storage: cryptsetup, lvm2
- Hardware: linux-firmware, tpm2-tools
- chrony
- Shell tools: vim-minimal, less, tar

NetworkManager, sshd, chronyd and nftables are enabled.

## Derive

**You can't log in yet.** There are no users and root is locked.

Follow the steps in our [derive repo](https://github.com/kheepercom/derive) to add a login user and launch or follow the quick start below.

## Quick Start

As a quicker alternative to deriving an image, you can use a special image name to inject an admin user with your GitHub authorized_keys.
`us.kheeper.com/bootc/gh/<user>/server:45`

That injects the keys from `https://github.com/<user>.keys` into the image on pull.
The image can be pulled without credentials.

Generate a script from https://kheeper.com/bootc-quickstart to launch on GCP, AWS or bare metal.
For example, GitHub user "mandi-bt" launching an instance on GCP would look like this.

```sh
gcloud compute instances create my-server \
  --image-project kheeper --image-family bootc-quickstart \
  --machine-type c4-standard-2 \
  --boot-disk-size 20GB \
  --metadata 'bootc-image=us.kheeper.com/bootc/gh/mandi-bt/server:45'
```

Then you can `ssh admin@<IP>`.

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
You can add additional packages in your derived images or start from these instead:

[`quay.io/fedora/fedora-iot:45`](https://quay.io/repository/fedora/fedora-iot) is Fedora's official IoT bootc image, built on the `minimal-plus` set. It adds firewalld, greenboot (health checks after an upgrade, with automatic rollback), ignition (first-boot provisioning), clevis with TPM2 (automatic LUKS unlock), FDO onboarding, usbguard, and Wi-Fi and WWAN support.

[`quay.io/fedora/fedora-bootc:45`](https://quay.io/repository/fedora/fedora-bootc) is the standard image: a general-purpose base, roughly a headless server install, with automatic updates on.
