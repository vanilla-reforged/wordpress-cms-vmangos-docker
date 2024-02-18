# lazycms-vmangos-docker

### Before you dive in



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

#### Connect to your IP or website address to do the basic wordpress setup:

- Use the sql user and database name from your .env file.
- The database hostname is wordpress_database.

#### Edit your wp-config.php file, so wordpress can be reached behind a reverse proxy:

Open your wp-config.php file located in var/www/html in your lazycms-vmangos-docker directory.

Add this code at the beginning of the file right after "<?php":

```sh
if (strpos($_SERVER['HTTP_X_FORWARDED_PROTO'], 'https') !== false) {
    $_SERVER['HTTPS'] = 'on';
}
```

#### Install WPCode Plugin & create Page to be used for Registration:

Use official Wordpress documentation if you need help with this.

### PHP CODE REGISTRATION FORM

#### Used themes and plugins for https://www.vanillareforged.org/:
Theme: Twenty Seventeen

Plugins: Options Twenty Seventeen, Updraftplus, WPCode

## Vanilla Reforged Links

Find and join us on the web https://vanillareforged.org/

Support our efforts on Patreon https://www.patreon.com/vanillareforged
