# postgres

[PostgreSQL](https://www.postgresql.org) operating system image on a [minimal Fedora server](../server).

## Release

`us.kheeper.com/bootc/postgres` and `eu.kheeper.com/bootc/postgres`, rebuilt when Fedora or PostgreSQL changes:

- `:18` follows PostgreSQL 18 minor releases from the [PostgreSQL yum repository](https://yum.postgresql.org).

## Launch

Follow the steps in our [derive repo](https://github.com/kheepercom/derive) to add a login user and launch this image on your preferred cloud or bare metal platform.

PostgreSQL starts on first boot with a new cluster in `/var/lib/pgsql/18/data` and listens only on localhost and its unix socket.
Its config lives in the image: `/etc/postgresql/postgresql.conf` includes `/etc/postgresql/conf.d/*.conf`, and `pg_hba.conf` allows peer and local password logins.

To accept TLS connections from other hosts, add a config file, a `pg_hba.conf`, a certificate and a firewall rule in your derived image:

```Containerfile
FROM us.kheeper.com/bootc/postgres:18

COPY remote.conf /etc/postgresql/conf.d/remote.conf
COPY pg_hba.conf /etc/postgresql/pg_hba.conf
COPY --chown=root:postgres --chmod=0640 cert.pem key.pem /etc/postgresql/tls/
RUN echo 'add element inet server tcp_ports { 5432 }' > /usr/lib/nftables/server.d/postgres.nft

# Add a user with an SSH authorized key, etc...
```

```ini
# remote.conf
listen_addresses = '*'
ssl = on
ssl_cert_file = '/etc/postgresql/tls/cert.pem'
ssl_key_file = '/etc/postgresql/tls/key.pem'
```

```
# pg_hba.conf
local    all  all             peer
hostssl  all  all  0.0.0.0/0  scram-sha-256
```

## Alternative Quick Start

As a quicker alternative to deriving an image, you can use a special image name to inject an admin user with your GitHub authorized_keys: `us.kheeper.com/bootc/gh/<user>/postgres:18`.

That injects the keys from `https://github.com/<user>.keys` into the image on pull.
The image can be pulled without credentials.

Generate a script from https://kheeper.com/bootc-quickstart to launch on GCP, AWS or bare metal.
For example, launching an instance on GCP looks like this, with your GitHub username in place of `<user>`.

```sh
gcloud compute instances create my-postgres-server \
  --image-project kheeper --image-family bootc-quickstart \
  --machine-type c4-standard-2 \
  --boot-disk-size 20GB \
  --metadata 'bootc-image=us.kheeper.com/bootc/gh/<user>/postgres:18'
```

Then you can `ssh admin@<IP>` and run `sudo -u postgres psql`.

No certificate? [postgres/easy](easy) accepts TLS connections at its public IP with a Let's Encrypt IP certificate.

## Notes

The cluster is initialized with the builtin `C.UTF-8` locale provider, so Fedora's glibc updates never change collation order under existing indexes.

The `postgres` superuser has no password; use `sudo -u postgres psql` on the host.

Upgrading to a new major version is a new tag and needs `pg_upgrade`; the image never does it for you.
