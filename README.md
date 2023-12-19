# lazycms-vmangos-docker

### Dependencies

+ docker
+ docker-compose

### Preface

This a simple CMS solution for vanilla wow servers using Docker and Drupal, heavily drawing form this guide:

https://www.digitalocean.com/community/tutorials/how-to-install-drupal-with-docker-compose

### Instructions lazycms

First, clone the repository and move into it.

```sh
user@local:~$ git clone https://github.com/vanilla-reforged/lazycms-vmangos-docker
user@local:~$ cd lazycms-vmangos-docker
```

At this point, you have to adjust the variables in the files:
- `./.env`
- `./vol/ngix-conf/nginx.conf` (server_name sections and ssl certificate path)
- `./vol/ngix-conf-setup/nginx.conf` (server_name sections)
- `./docker-compose.yml` (service drupal_certbot, command section)
- `./docker-compose-setup.yml`
to fit your desired setup. 

then execute the setup docker compose file

`docker compose -f docker-compose-setup.yml up -d`

Check the status of the services using the docker-compose ps command:

`docker compose ps`

We will see the mysql, drupal, and webserver services with a State of Up, while certbot will be exited with a 0 status message:

If you see anything other than Up in the State column for the mysql, drupal, or webserver services, or an exit status other than 0 for the certbot container, be sure to check the service logs with the docker-compose logs command:

`docker compose logs service_name`

We can now check that our certificates mounted on the webserver container using the docker-compose exec command:

`docker-compose exec webserver ls -la /etc/letsencrypt/live`

This will show the following directory

`drwxr-xr-x    2 root     root          4096 Oct  5 09:15 your_domain`

Then shutdown and restart your environment by using

`docker compose down`
`docker compose up -d`

## Vanilla Reforged Links

Find and join us on the web https://vanillareforged.org/en/

Support our efforts on Patreon https://www.patreon.com/vanillareforged

## Links

[vmangos]: https://github.com/vmangos/core
