# lazycms-vmangos-docker

### Dependencies

+ docker
+ docker compose 2

### Preface

This a simple CMS solution for vanilla wow servers using docker and wordpress.

Result: https://vanillareforged.org/

### Instructions lazycms
### Docker Setup

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

### Wordpress basic setup

#### Connect to your IP or website address to start the wordpress installation: 
- Use the sql user and database name from your .env file.
- The database hostname is wordpress_database.


#### Install theme and plugins:
Theme: Twenty Seventeen

Plugins: Options Twenty Seventeen, Updraftplus, Really Simple SSL, WPCode


#### SSL


#### Setup Updraftplus Backup and do an initial backup:


#### Set up registration page:
- Create new page to be used as registration page.



### PHP CODE REGISTRATION FORM



## Vanilla Reforged Links

Find and join us on the web https://vanillareforged.org/

Support our efforts on Patreon https://www.patreon.com/vanillareforged
