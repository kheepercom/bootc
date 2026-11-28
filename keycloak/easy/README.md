# keycloak easy

[keycloak](../) that configures itself on first boot: no domain name, database or config file needed.

`us.kheeper.com/bootc/keycloak/easy` and `eu.kheeper.com/bootc/keycloak/easy`, tagged like keycloak.

This image is intended for getting started quickly until you have your own domain or your own opinions on how to operate Keycloak.

## Quick Start

`us.kheeper.com/bootc/gh/<user>/keycloak/easy:26` adds an admin user with the keys from `https://github.com/<user>.keys`.

Generate a script from https://kheeper.com/bootc-quickstart to launch on GCP, AWS or bare metal.
For example, launching an instance on GCP looks like this, with your GitHub username in place of `<user>`.

```sh
gcloud compute instances create keycloak-easy \
  --image-project kheeper --image-family bootc-quickstart \
  --machine-type c4-standard-2 \
  --boot-disk-size 20GB \
  --tags keycloak \
  --metadata 'bootc-image=us.kheeper.com/bootc/gh/<user>/keycloak/easy:26'

gcloud compute firewall-rules create keycloak --target-tags keycloak \
  --allow tcp:80,tcp:443
```

Once it's up, `ssh admin@<IP>`, get the bootstrap admin's password and log in at `https://<IP>/admin` as `admin`:

```sh
sudo grep bootstrap-admin-password /etc/keycloak/keycloak.conf
```

The bootstrap admin is temporary; create a permanent admin in the master realm and delete it.

## How it works

On first boot, `ip-cert.service` looks up the host's public IPv4 address with ifconfig.co and gets a Let's Encrypt certificate for it over port 80.
If `/etc/keycloak/keycloak.conf` doesn't exist, `keycloak-easy.service` creates a `keycloak` database in the local PostgreSQL and writes a config serving `https://<IP>`.

Let's Encrypt IP certificates last 6 days. `ip-cert.timer` renews at half-life, and Keycloak picks up the new certificate within an hour without restarting.

Edit `/etc/keycloak/keycloak.conf` freely; it's never rewritten.
