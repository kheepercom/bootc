# pihole

[Pi-hole](https://pi-hole.net) operating system image on a [minimal Fedora server](../server).

## Release

`us.kheeper.com/bootc/pihole` and `eu.kheeper.com/bootc/pihole`, rebuilt when Fedora or Pi-hole changes:

- `:6` follows Pi-hole v6 releases.

## Launch

Follow the steps in our [derive repo](https://github.com/kheepercom/derive) to add a login user and launch this image on your preferred cloud or bare metal platform.

Pi-hole starts on first boot and answers DNS queries only from networks the host is directly connected to, so it is never an open resolver.
It forwards to Google's DNS until you choose other upstream servers.

Set the web interface password and other settings with the `pihole` CLI or at `https://<IP>/admin`:

```sh
sudo podman exec -it pihole pihole setpassword
```

Until then, it uses a random password printed in `sudo podman logs pihole`.

To serve networks that aren't directly connected, such as a tailnet or other instances on a cloud like GCE that gives each one a /32, permit all origins and restrict port 53 with your cloud's firewall instead:

```sh
sudo podman exec pihole pihole-FTL --config dns.listeningMode ALL
```

## Alternative Quick Start

As a quicker alternative to deriving an image, you can use a special image name to inject an admin user with your GitHub authorized_keys: `us.kheeper.com/bootc/gh/<user>/pihole:6`.

That injects the keys from `https://github.com/<user>.keys` into the image on pull.
The image can be pulled without credentials.

Generate a script from https://kheeper.com/bootc-quickstart to launch on GCP, AWS or bare metal.
For example, launching an instance on GCP looks like this, with your GitHub username in place of `<user>`.

```sh
gcloud compute instances create my-pihole-server \
  --image-project kheeper --image-family bootc-quickstart \
  --machine-type e2-small \
  --boot-disk-size 20GB \
  --metadata 'bootc-image=us.kheeper.com/bootc/gh/<user>/pihole:6'
```

Then you can `ssh admin@<IP>` and point your network's DHCP or clients at it for DNS.

## Notes

The image runs upstream's container (pinned by digest) with host networking. It is a [logically bound image](https://bootc-dev.github.io/bootc/logically-bound-images.html), so bootc pulls it before rebooting into an update.

State lives in `/var/lib/pihole`, mounted at `/etc/pihole` in the container.

systemd-resolved's stub listener is off so Pi-hole can listen on port 53, and the host resolves through its upstream servers directly.

The web interface uses a self-signed certificate until you put your own in `/var/lib/pihole/tls.pem` and set `webserver.domain`.

The firewall opens 53/tcp, 53/udp and 443/tcp.
