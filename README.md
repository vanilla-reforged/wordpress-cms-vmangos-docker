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

At this point, you have to adjust the variables in the files:
- `./.env`
- `./vol/nginx.conf` (server_name sections and ssl certificate path)
- `./docker-compose.yml` (service drupal_certbot, command section)
to fit your desired setup. 

## Vanilla Reforged Links

Find and join us on the web https://vanillareforged.org/en/

Support our efforts on Patreon https://www.patreon.com/vanillareforged

## Links

[vmangos]: https://github.com/vmangos/core
