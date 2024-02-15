# lazycms-vmangos-docker

### Dependencies

+ docker
+ docker compose 2

### Preface

This a simple CMS solution for vanilla wow servers using docker and wordpress.

Result: https://vanillareforged.org/

### Instructions lazycms

First, clone the repository and move into it.

```sh
git clone https://github.com/vanilla-reforged/lazycms-vmangos-docker/
cd lazycms-vmangos-docker
```

At this point, you have to adjust the `./.env` for your desired setup.

Then start your environment with:

```sh
docker compose up -d
```

Then connect to your IP or website name to do the basic Setup.

Install theme and Plugins as necessary

Dont forget to outcomment Port 80 in your docker-compose.yml once you have installed your SSL Certificate

```sh
   ports:
 #     - 80:80
```

### WORDPRESS THEME & PLUGINS

Theme: Twenty Seventeen

Plugins: Options Twenty Seventeen, WP Encryption, WPCode

### PHP CODE REGISTRATION FORM



## Vanilla Reforged Links

Find and join us on the web https://vanillareforged.org/

Support our efforts on Patreon https://www.patreon.com/vanillareforged

## Links

- [vmangos](https://github.com/vmangos/core)
- [Drupal]
- [Drupal with Docker Compose](https://www.digitalocean.com/community/tutorials/how-to-install-drupal-with-docker-compose)
- [Webform Remote Handlers](https://www.drupal.org/project/webform_remote_handlers)
