# bootc

Bootable OS images for open source software, rebuilt nightly, updated automatically to new releases, and tested on real instances before each release.

| Image | |
| ----- | - |
| [server](server) | Minimal Fedora server the others build on |
| [forgejo](forgejo) ([easy](forgejo/easy), [runner](forgejo/runner)) | Git forge and Actions runner |
| [grafana](grafana) ([easy](grafana/easy)) | Dashboards |
| [headscale](headscale) ([easy](headscale/easy)) | Tailscale control server |
| [keycloak](keycloak) ([easy](keycloak/easy)) | Identity provider |
| [pihole](pihole) | DNS sinkhole |
| [postgres](postgres) ([easy](postgres/easy)) | PostgreSQL |

The easy images configure themselves on first boot to serve the host's public IP with a Let's Encrypt IP certificate, for getting started without a domain name.
