# lib

Pieces shared by the `*/easy` images, passed to their builds with `--build-context lib=lib`.

- `ipcert/install` adds lego and `ip-cert.service`, which gets and renews a Let's Encrypt certificate for the host's public IPv4 address. Each image provides `/usr/libexec/ip-cert-deploy` to install the certificate for its service.
- `postgresql/install` adds Fedora's PostgreSQL, initialized on first boot for local use.
