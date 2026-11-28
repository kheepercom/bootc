# forgejo easy

[forgejo](../) that configures itself on first boot: no domain name, database or config file needed.

`us.kheeper.com/bootc/forgejo/easy` and `eu.kheeper.com/bootc/forgejo/easy`, tagged like forgejo.

This image is intended for getting started quickly until you have your own domain or your own opinions on how to operate Forgejo.

## Quick Start

`us.kheeper.com/bootc/gh/<user>/forgejo/easy:17` adds an admin user with the keys from `https://github.com/<user>.keys`.

Generate a script from https://kheeper.com/bootc-quickstart to launch on GCP, AWS or bare metal.
For example, launching an instance on GCP looks like this, with your GitHub username in place of `<user>`.

```sh
gcloud compute instances create forgejo-easy \
  --image-project kheeper --image-family bootc-quickstart \
  --machine-type c4-standard-2 \
  --boot-disk-size 20GB \
  --tags forgejo \
  --metadata 'bootc-image=us.kheeper.com/bootc/gh/<user>/forgejo/easy:17'

gcloud compute firewall-rules create forgejo --target-tags forgejo \
  --allow tcp:80,tcp:443
```

Once it's up, `ssh admin@<IP>`, get the password for Forgejo's `root` user and log in at `https://<IP>`, where you'll be asked to change it:

```sh
sudo cat /etc/forgejo/root-password
```

Clone over SSH with `git clone git@<IP>:<owner>/<repo>.git` once you've added a key to your Forgejo account.

## How it works

On first boot, `ip-cert.service` looks up the host's public IPv4 address with ifconfig.co and gets a Let's Encrypt certificate for it over port 80.
If `/etc/forgejo/app.ini` doesn't exist, `forgejo-easy.service` creates a `forgejo` database in the local PostgreSQL, writes a config serving `https://<IP>` with registration disabled, and creates the `root` admin.

Let's Encrypt IP certificates last 6 days. `ip-cert.timer` renews at half-life and restarts Forgejo.

Edit `/etc/forgejo/app.ini` freely; it's never rewritten.
