# keycloak

[Keycloak](https://www.keycloak.org) operating system image on a [minimal Fedora server](../server).

## Release

`us.kheeper.com/bootc/keycloak` and `eu.kheeper.com/bootc/keycloak`, rebuilt when Fedora or Keycloak changes:

- `:26.8` follows Keycloak patch releases.
- `:26` also follows minor releases, which may migrate the database.

## Launch

Follow the steps in our [derive repo](https://github.com/kheepercom/derive) to add a login user and launch this image on your preferred cloud or bare metal platform.

Keycloak will not start until `/etc/keycloak/keycloak.conf` exists.
Either add your config to your derived image or SSH in after boot to write it (and then run `sudo systemctl start keycloak`).

```Containerfile
FROM us.kheeper.com/bootc/keycloak:26

COPY keycloak.conf /etc/keycloak/keycloak.conf
COPY cert.pem key.pem /etc/keycloak/tls/

# Add a user with an SSH authorized key, etc...
```

Keycloak needs a PostgreSQL database:

```ini
hostname=https://auth.example.com
https-port=443
https-certificate-file=/etc/keycloak/tls/cert.pem
https-certificate-key-file=/etc/keycloak/tls/key.pem
db-url=jdbc:postgresql://db.example.com/keycloak?sslmode=verify-full
db-username=keycloak
db-password=<password>
bootstrap-admin-username=admin
bootstrap-admin-password=<password>
```

The bootstrap admin is created on first start; replace it with a permanent one in the admin console.
See Keycloak's [server guides](https://www.keycloak.org/guides#server) for everything else.

## Alternative Quick Start

As a quicker alternative to deriving an image, you can use a special image name to inject an admin user with your GitHub authorized_keys: `us.kheeper.com/bootc/gh/<user>/keycloak:26`.

That injects the keys from `https://github.com/<user>.keys` into the image on pull.
The image can be pulled without credentials.

Generate a script from https://kheeper.com/bootc-quickstart to launch on GCP, AWS or bare metal.
For example, launching an instance on GCP looks like this, with your GitHub username in place of `<user>`.

```sh
gcloud compute instances create my-keycloak-server \
  --image-project kheeper --image-family bootc-quickstart \
  --machine-type c4-standard-2 \
  --boot-disk-size 20GB \
  --tags keycloak \
  --metadata 'bootc-image=us.kheeper.com/bootc/gh/<user>/keycloak:26'

gcloud compute firewall-rules create keycloak --target-tags keycloak \
  --allow tcp:443
```

Then you can `ssh admin@<IP>`, write /etc/keycloak/keycloak.conf and run `sudo systemctl start keycloak`.

No domain name or database? [keycloak/easy](easy) configures itself for `https://<IP>` with a local PostgreSQL and a Let's Encrypt IP certificate.

## Notes

The image includes the official release (checksum pinned) in `/usr/lib/keycloak`, prebuilt for PostgreSQL with a local cache and health and metrics on the management port (9000, not opened), and runs it as the `keycloak` user with OpenJDK 25.

Keycloak rereads its certificate files hourly, so renewing them needs no restart.

Everything in `/etc/keycloak` is made readable by the `keycloak` group at boot, so keep secrets there rather than in world-readable files.

The firewall opens 443/tcp.
