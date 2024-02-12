# lazycms-vmangos-docker

### Dependencies

+ docker
+ docker compose 2

### Preface

This a simple CMS solution for vanilla wow servers using Docker and Drupal, heavily drawing form this guide:

### Instructions lazycms

FW rules: 80/443

First, clone the repository and move into it.

```sh
git clone https://github.com/vanilla-reforged/lazycms-vmangos-docker/
cd lazycms-vmangos-docker
```
At this point, you have to adjust the `./.env` for your desired setup.

Start your environment with:

```sh
docker compose up -d
```

Attach to your Drupal container from the Docker host:

```sh
docker exec -it drupal /bin/bash
```

Copy the default.settings file to the location where drupal expects it:

```sh
mkdir -p /opt/drupal/web/sites/default/files/ && cp /opt/drupal/web/core/assets/scaffold/files/default.settings.php /opt/drupal/web/sites/default/files/settings.php
```

Exit the container

```sh
exit
```

Then connect to your web address or IP to install Drupal.

-> execute script 01 to install SOAP extension

-> certbot

## Vanilla Reforged Links

Find and join us on the web https://vanillareforged.org/

Support our efforts on Patreon https://www.patreon.com/vanillareforged

## Links

- [vmangos](https://github.com/vmangos/core)
- [Drupal]
- [Drupal with Docker Compose](https://www.digitalocean.com/community/tutorials/how-to-install-drupal-with-docker-compose)
- [Webform Remote Handlers](https://www.drupal.org/project/webform_remote_handlers)
