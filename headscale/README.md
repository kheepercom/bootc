# headscale

[Headscale](https://headscale.net) operating system image on a [minimal Fedora server](../server).

## Release

`us.kheeper.com/bootc/headscale` and `eu.kheeper.com/bootc/headscale`, rebuilt when Fedora or headscale changes:

- `:0.29` follows headscale patch releases.
- `:0` also follows minor releases, which may change the config or migrate the database.
- `:0.29.7` stays on that release but still gets OS updates until the next one.
- `:0.29.7-YYYYMMDD` never changes.

## Launch

Follow the steps in our [derive repo](https://github.com/kheepercom/derive) to add a login user and launch this image on your preferred cloud or bare metal platform.

Headscale will not start until `/etc/headscale/config.yaml` exists.
Either add your config to your derived image or SSH in after boot to write it (and then run `sudo systemctl start headscale`).

```Containerfile
FROM us.kheeper.com/bootc/headscale:0.29

COPY config.yaml /etc/headscale/config.yaml

# Add a user with an SSH authorized key, etc...
```

Start from upstream's [config-example.yaml](https://github.com/juanfont/headscale/blob/v0.29.7/config-example.yaml) and set at least `server_url`, `listen_addr: 0.0.0.0:443` and either `tls_letsencrypt_hostname` or `tls_cert_path`/`tls_key_path`.

## Alternative Quick Start

As a quicker alternative to deriving an image, you can use a special image name to inject an admin user with your GitHub authorized_keys: `us.kheeper.com/bootc/gh/<user>/headscale:0`.

That injects the keys from `https://github.com/<user>.keys` into the image on pull.
The image can be pulled without credentials.

Generate a script from https://kheeper.com/bootc-quickstart to launch on GCP, AWS or bare metal.
For example, launching an instance on GCP looks like this, with your GitHub username in place of `<user>`.

```sh
gcloud compute instances create my-headscale-server \
  --image-project kheeper --image-family bootc-quickstart \
  --machine-type c4-standard-2 \
  --boot-disk-size 20GB \
  --tags headscale \
  --metadata 'bootc-image=us.kheeper.com/bootc/gh/<user>/headscale:0'

gcloud compute firewall-rules create headscale --target-tags headscale \
  --allow tcp:80,tcp:443,udp:3478
```

Then you can `ssh admin@<IP>`, write /etc/headscale/config.yaml, run `sudo systemctl start headscale`, and add users and nodes with `sudo headscale`.

No domain name? [headscale/easy](easy) configures itself for `https://<IP>` with a Let's Encrypt IP certificate.

## Notes

The image includes the official release binary (checksum pinned) and upstream's hardened systemd unit modified to run with a `DynamicUser`.

State lives in `/var/lib/headscale`, a symlink systemd manages into `/var/lib/private`.

The `headscale` CLI must be run with sudo since it talks to the server over a socket owned by the headscale service.

The firewall opens 80/tcp (Let's Encrypt HTTP-01), 443/tcp and 3478/udp (embedded DERP's STUN).
