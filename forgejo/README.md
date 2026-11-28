# forgejo

[Forgejo](https://forgejo.org) operating system image on a [minimal Fedora server](../server).

## Release

`us.kheeper.com/bootc/forgejo` and `eu.kheeper.com/bootc/forgejo`, rebuilt when Fedora or Forgejo changes:

- `:17` follows patch releases.

Forgejo's major releases migrate the database and may change the config, so each one is a new tag.

## Launch

Follow the steps in our [derive repo](https://github.com/kheepercom/derive) to add a login user and launch this image on your preferred cloud or bare metal platform.

Forgejo will not start until `/etc/forgejo/app.ini` exists.
Either add your config to your derived image or SSH in after boot to write it (and then run `sudo systemctl start forgejo`).

```Containerfile
FROM us.kheeper.com/bootc/forgejo:17

COPY app.ini /etc/forgejo/app.ini

# Add a user with an SSH authorized key, etc...
```

Forgejo needs a PostgreSQL database. For example, with Forgejo's built-in Let's Encrypt support:

```ini
[server]
DOMAIN = git.example.com
ROOT_URL = https://git.example.com/
PROTOCOL = https
HTTP_PORT = 443
ENABLE_ACME = true
ACME_ACCEPTTOS = true
ACME_EMAIL = you@example.com

[database]
DB_TYPE = postgres
HOST = db.example.com:5432
NAME = forgejo
USER = forgejo
PASSWD = <password>
SSL_MODE = verify-full

[security]
INSTALL_LOCK = true
```

See Forgejo's [config cheat sheet](https://forgejo.org/docs/latest/admin/config-cheat-sheet/) for everything else.
Create the first admin with `sudo -u git forgejo --config /etc/forgejo/app.ini admin user create --admin ...`.

## Alternative Quick Start

As a quicker alternative to deriving an image, you can use a special image name to inject an admin user with your GitHub authorized_keys: `us.kheeper.com/bootc/gh/<user>/forgejo:17`.

That injects the keys from `https://github.com/<user>.keys` into the image on pull.
The image can be pulled without credentials.

Generate a script from https://kheeper.com/bootc-quickstart to launch on GCP, AWS or bare metal.
For example, launching an instance on GCP looks like this, with your GitHub username in place of `<user>`.

```sh
gcloud compute instances create my-forgejo-server \
  --image-project kheeper --image-family bootc-quickstart \
  --machine-type c4-standard-2 \
  --boot-disk-size 20GB \
  --tags forgejo \
  --metadata 'bootc-image=us.kheeper.com/bootc/gh/<user>/forgejo:17'

gcloud compute firewall-rules create forgejo --target-tags forgejo \
  --allow tcp:80,tcp:443
```

Then you can `ssh admin@<IP>`, write /etc/forgejo/app.ini and run `sudo systemctl start forgejo`.

Run Actions jobs on [forgejo/runner](runner).

No domain name or database? [forgejo/easy](easy) configures itself for `https://<IP>` with a local PostgreSQL and a Let's Encrypt IP certificate.

## Notes

The image includes the official release binary (checksum pinned) and git, and runs Forgejo as the `git` user.

Git over SSH goes through the host's sshd on port 22, alongside your own logins: Forgejo keeps `/var/lib/forgejo/.ssh/authorized_keys` up to date, so leave `START_SSH_SERVER` off.

State lives in `/var/lib/forgejo`. Forgejo saves any secrets it has to generate into `/etc/forgejo/app.ini`, which then keeps later images' `app.ini` from replacing it, so put `SECRET_KEY`, `INTERNAL_TOKEN`, `JWT_SECRET` and `LFS_JWT_SECRET` from `forgejo generate secret` in yours.

The firewall opens 80/tcp (Let's Encrypt HTTP-01) and 443/tcp.
