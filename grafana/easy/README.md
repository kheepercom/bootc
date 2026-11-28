# grafana easy

[grafana](../) that configures itself on first boot: no domain name and no config file needed.

`us.kheeper.com/bootc/grafana/easy` and `eu.kheeper.com/bootc/grafana/easy`, tagged like grafana.

This image is intended for getting started quickly until you have your own domain or your own opinions on how to operate Grafana.

## Quick Start

`us.kheeper.com/bootc/gh/<user>/grafana/easy:13` adds an admin user with the keys from `https://github.com/<user>.keys`.

Generate a script from https://kheeper.com/bootc-quickstart to launch on GCP, AWS or bare metal.
For example, launching an instance on GCP looks like this, with your GitHub username in place of `<user>`.

```sh
gcloud compute instances create grafana-easy \
  --image-project kheeper --image-family bootc-quickstart \
  --machine-type c4-standard-2 \
  --boot-disk-size 20GB \
  --tags grafana \
  --metadata 'bootc-image=us.kheeper.com/bootc/gh/<user>/grafana/easy:13'

gcloud compute firewall-rules create grafana --target-tags grafana \
  --allow tcp:80,tcp:443
```

Once it's up, `ssh admin@<IP>`, get the admin password and log in at `https://<IP>` as `admin`:

```sh
sudo grep admin_password /etc/grafana/grafana.ini
```

## How it works

On first boot, `ip-cert.service` looks up the host's public IPv4 address with ifconfig.co and gets a Let's Encrypt certificate for it over port 80.
If `/etc/grafana/grafana.ini` doesn't exist, `grafana-easy.service` writes one serving `https://<IP>` with a random admin password and secret key.

Let's Encrypt IP certificates last 6 days. `ip-cert.timer` renews at half-life and restarts Grafana.

Edit `/etc/grafana/grafana.ini` freely; it's never rewritten. Changing `admin_password` there has no effect after the first start; change it in Grafana.
