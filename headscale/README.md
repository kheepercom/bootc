# headscale

[Headscale](https://headscale.net) on [server](../server): the official release binary (checksum pinned), upstream's hardened systemd unit, run as a `DynamicUser`. State lives in `/var/lib/headscale`, a symlink systemd manages into `/var/lib/private`. Run the `headscale` CLI with sudo.

The firewall opens 80/tcp (Let's Encrypt HTTP-01), 443/tcp and 3478/udp (embedded DERP's STUN).

## Release

`us.kheeper.com/bootc/headscale:0.29.7` and `eu.kheeper.com/bootc/headscale:0.29.7`.

## Configure

Headscale starts once `/etc/headscale/config.yaml` exists. Derive with your config and an SSH key:

```Containerfile
FROM us.kheeper.com/bootc/headscale:0.29.7
COPY config.yaml /etc/headscale/config.yaml
COPY root.keys /usr/ssh/root.keys
RUN echo 'AuthorizedKeysFile /usr/ssh/%u.keys .ssh/authorized_keys' > /etc/ssh/sshd_config.d/30-auth-system.conf
```

Start from upstream's [config-example.yaml](https://github.com/juanfont/headscale/blob/v0.29.7/config-example.yaml) and set at least `server_url`, `listen_addr: 0.0.0.0:443` and either `tls_letsencrypt_hostname` or `tls_cert_path`/`tls_key_path`.
