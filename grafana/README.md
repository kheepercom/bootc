# grafana

[Grafana](https://grafana.com/oss/grafana/) operating system image on a [minimal Fedora server](../server).

## Release

`us.kheeper.com/bootc/grafana` and `eu.kheeper.com/bootc/grafana`, rebuilt when Fedora or Grafana changes:

- `:13.3` follows Grafana patch releases.
- `:13` also follows minor releases, which may migrate the database.

## Launch

Follow the steps in our [derive repo](https://github.com/kheepercom/derive) to add a login user and launch this image on your preferred cloud or bare metal platform.

Grafana will not start until `/etc/grafana/grafana.ini` exists.
Either add your config to your derived image or SSH in after boot to write it (and then run `sudo systemctl start grafana`).

```Containerfile
FROM us.kheeper.com/bootc/grafana:13

COPY grafana.ini /etc/grafana/grafana.ini
COPY cert.pem key.pem /etc/grafana/tls/
COPY provisioning/ /etc/grafana/provisioning/

# Add a user with an SSH authorized key, etc...
```

```ini
[server]
protocol = https
http_port = 443
domain = grafana.example.com
cert_file = /etc/grafana/tls/cert.pem
cert_key = /etc/grafana/tls/key.pem

[security]
admin_password = <password>
secret_key = <random string>
```

See Grafana's [configuration docs](https://grafana.com/docs/grafana/latest/setup-grafana/configure-grafana/) for everything else, and [provisioning](https://grafana.com/docs/grafana/latest/administration/provisioning/) for data sources and dashboards in `/etc/grafana/provisioning`.

## Alternative Quick Start

As a quicker alternative to deriving an image, you can use a special image name to inject an admin user with your GitHub authorized_keys: `us.kheeper.com/bootc/gh/<user>/grafana:13`.

That injects the keys from `https://github.com/<user>.keys` into the image on pull.
The image can be pulled without credentials.

Generate a script from https://kheeper.com/bootc-quickstart to launch on GCP, AWS or bare metal.
For example, launching an instance on GCP looks like this, with your GitHub username in place of `<user>`.

```sh
gcloud compute instances create my-grafana-server \
  --image-project kheeper --image-family bootc-quickstart \
  --machine-type c4-standard-2 \
  --boot-disk-size 20GB \
  --tags grafana \
  --metadata 'bootc-image=us.kheeper.com/bootc/gh/<user>/grafana:13'

gcloud compute firewall-rules create grafana --target-tags grafana \
  --allow tcp:443
```

Then you can `ssh admin@<IP>`, write /etc/grafana/grafana.ini and run `sudo systemctl start grafana`.

No domain name? [grafana/easy](easy) configures itself for `https://<IP>` with a Let's Encrypt IP certificate.

## Notes

The image includes the official OSS release (checksum pinned) in `/usr/lib/grafana`, run as the `grafana` user.

State lives in `/var/lib/grafana`, including the default SQLite database; set `[database]` to use PostgreSQL or MySQL instead.

Everything in `/etc/grafana` is made readable by the `grafana` group at boot, so keep secrets there rather than in world-readable files.

The firewall opens 443/tcp.
