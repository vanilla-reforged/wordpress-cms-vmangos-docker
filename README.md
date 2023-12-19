# lazycms-vmangos-docker

### Dependencies

+ docker
+ docker-compose

### Preface

This a simple CMS solution for vanilla wow servers.

Using Docker and Drupal, heavily drawing form this guide:

https://www.digitalocean.com/community/tutorials/how-to-install-drupal-with-docker-compose

### Instructions lazycms

First, clone the repository and move into it.

```sh
user@local:~$ git clone https://github.com/vanilla-reforged/lazycms-vmangos-docker
user@local:~$ cd lazycms-vmangos-docker
```

At this point, you have to adjust the docker configuration file `./.env`, the NGINX configuration file `./vol/nginx.conf` and the command section for your certbot in the docker-compose.yml to fit your desired setup. 

then start your environment

```sh
user@local:lazycms-vmangos-docker$ docker compose up -d
```
Now connect over the browser to 8080 / 8081 to start the setup.

## Vanilla Reforged Links

Find and join us on the web https://vanillareforged.org/en/

Support our efforts on Patreon https://www.patreon.com/vanillareforged

## Links

[vmangos]: https://github.com/vmangos/core
