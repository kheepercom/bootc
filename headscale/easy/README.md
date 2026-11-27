# headscale easy

[headscale](../) that configures itself on first boot: no domain name and no config file needed.

`us.kheeper.com/bootc/headscale/easy` and `eu.kheeper.com/bootc/headscale/easy`, tagged like headscale.

This image is intended for getting started quickly until you have your own domain or your own opinions on how to operate headscale.

## Quick Start

`us.kheeper.com/bootc/gh/<user>/headscale/easy:0` adds an admin user with the keys from `https://github.com/<user>.keys`.

Generate a script from https://kheeper.com/bootc-quickstart to launch on GCP, AWS or bare metal.
For example, launching an instance on GCP looks like this, with your GitHub username in place of `<user>`.

```sh
gcloud compute instances create headscale-easy \
  --image-project kheeper --image-family bootc-quickstart \
  --machine-type c4-standard-2 \
  --boot-disk-size 20GB \
  --tags headscale \
  --metadata 'bootc-image=us.kheeper.com/bootc/gh/<user>/headscale/easy:0'

gcloud compute firewall-rules create headscale --target-tags headscale \
  --allow tcp:80,tcp:443,udp:3478
```

Once it's up, `ssh admin@<IP>`, create a headscale user and a key, and log in from any device:

```sh
sudo headscale users create me
sudo headscale preauthkeys create --user 1

# From another machine
tailscale up --login-server https://<IP> --authkey <key>
```

## How it works

On first boot, if `/etc/headscale/config.yaml` doesn't exist, `headscale-easy.service` looks up the host's public IPv4 address with ifconfig.co, gets a Let's Encrypt certificate for it over port 80, and writes a config serving `https://<IP>` with headscale's embedded DERP as the only relay.

Let's Encrypt IP certificates last 6 days. `headscale-easy-cert.timer` renews at half-life and restarts headscale, which drops the control connection for a few seconds; existing tailscale connections carry on.

Edit `/etc/headscale/config.yaml` freely; it's never rewritten.
