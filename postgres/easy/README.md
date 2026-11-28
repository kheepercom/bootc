# postgres easy

[postgres](../) that accepts TLS connections at its public IP on first boot: no domain name, certificate or config needed.

`us.kheeper.com/bootc/postgres/easy` and `eu.kheeper.com/bootc/postgres/easy`, tagged like postgres.

This image is intended for getting started quickly until you have your own domain or your own opinions on how to operate PostgreSQL.

## Quick Start

`us.kheeper.com/bootc/gh/<user>/postgres/easy:18` adds an admin user with the keys from `https://github.com/<user>.keys`.

Generate a script from https://kheeper.com/bootc-quickstart to launch on GCP, AWS or bare metal.
For example, launching an instance on GCP looks like this, with your GitHub username in place of `<user>`.

```sh
gcloud compute instances create postgres-easy \
  --image-project kheeper --image-family bootc-quickstart \
  --machine-type c4-standard-2 \
  --boot-disk-size 20GB \
  --tags postgres \
  --metadata 'bootc-image=us.kheeper.com/bootc/gh/<user>/postgres/easy:18'

gcloud compute firewall-rules create postgres --target-tags postgres \
  --allow tcp:80,tcp:5432
```

Once it's up, `ssh admin@<IP>` and get the connection URL for the `app` database:

```sh
sudo cat /etc/postgresql/url

# From another machine (libpq 16 or later)
psql 'postgresql://app:<password>@<IP>/app?sslmode=verify-full&sslrootcert=system'
```

## How it works

On first boot, `ip-cert.service` looks up the host's public IPv4 address with ifconfig.co and gets a Let's Encrypt certificate for it over port 80.
PostgreSQL starts once the certificate is in `/etc/postgresql/tls`, and `postgres-easy.service` creates the `app` role and database with a random password.

Remote connections must use TLS and a password. The `postgres` superuser is only reachable with `sudo -u postgres psql` on the host.

Let's Encrypt IP certificates last 6 days. `ip-cert.timer` renews at half-life and reloads PostgreSQL, which keeps existing connections.
