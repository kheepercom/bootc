# forgejo runner

[Forgejo Runner](https://forgejo.org/docs/latest/admin/actions/) operating system image on a [minimal Fedora server](../../server), running Forgejo Actions jobs in rootless podman containers.

## Release

`us.kheeper.com/bootc/forgejo/runner` and `eu.kheeper.com/bootc/forgejo/runner`, rebuilt when Fedora or Forgejo Runner changes:

- `:13.2` follows Forgejo Runner patch releases.
- `:13` also follows minor releases.

## Launch

Follow the steps in our [derive repo](https://github.com/kheepercom/derive) to add a login user and launch this image on your preferred cloud or bare metal platform.

The runner will not start until `/etc/forgejo-runner/config.yml` exists.
Create a runner in Forgejo under **Site administration > Actions > Runners** (or an organization's, user's or repository's settings) and put its UUID and token in the config:

```Containerfile
FROM us.kheeper.com/bootc/forgejo/runner:13

COPY config.yml /etc/forgejo-runner/config.yml

# Add a user with an SSH authorized key, etc...
```

```yaml
server:
  connections:
    forgejo:
      url: https://git.example.com/
      uuid: <uuid>
      token: <token>
runner:
  capacity: 2
  labels:
    - docker:docker://data.forgejo.org/oci/node:22-bookworm
container:
  # Required: rootless podman can't exec into a container created with the default of 0.
  options: --oom-score-adj=1000
```

Jobs with `runs-on: docker` then run in that image. See the runner's [configuration docs](https://forgejo.org/docs/latest/admin/actions/configuration/) for everything else.

## Alternative Quick Start

As a quicker alternative to deriving an image, you can use a special image name to inject an admin user with your GitHub authorized_keys: `us.kheeper.com/bootc/gh/<user>/forgejo/runner:13`.

That injects the keys from `https://github.com/<user>.keys` into the image on pull.
The image can be pulled without credentials.

Generate a script from https://kheeper.com/bootc-quickstart to launch on GCP, AWS or bare metal.
For example, launching an instance on GCP looks like this, with your GitHub username in place of `<user>`.

```sh
gcloud compute instances create my-forgejo-runner \
  --image-project kheeper --image-family bootc-quickstart \
  --machine-type c4-standard-4 \
  --boot-disk-size 50GB \
  --metadata 'bootc-image=us.kheeper.com/bootc/gh/<user>/forgejo/runner:13'
```

Then you can `ssh admin@<IP>`, write /etc/forgejo-runner/config.yml and start the runner:

```sh
sudo systemctl --user -M runner@ restart forgejo-runner
sudo journalctl _SYSTEMD_USER_UNIT=forgejo-runner.service
```

## Notes

The image includes the official release binary (checksum pinned).
It runs as the `runner` user in that user's own systemd session, which starts at boot, along with the podman API socket the runner starts job containers through.
Jobs never get root on the host.

Job containers, images and caches live in `/var/lib/forgejo-runner`.

No ports are opened: the runner only connects out to Forgejo.
