# lazycms-vmangos-docker

### Dependencies

+ docker
+ docker-compose

### Preface

This a simple CMS to be used together with the vmangos-docker solution.

### Instructions lazycms

First, clone the repository and move into it.

```sh
user@local:~$ git clone https://github.com/vanilla-reforged/lazycms-vmangos-docker
user@local:~$ cd lazycms-vmangos-docker
```

At this point, you have to adjust the configuration file in `./.env` for your desired setup. 

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