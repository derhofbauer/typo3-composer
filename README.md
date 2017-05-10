# typo3-composer

This repository contains build instructions for a simple TYPO3 Composer Mode Docker image, bundeling the typo3_console extensions.

## Usage

This Container does not ship a database management system; I recommend to use `docker-compose` for this purpose.

You can use this image for programmatically setting up a TYPO3 Environment. Use a shell script for installing TYPO3 and importing a DB Dump:

```sh
$TYPO3CMS_PATH install:setup --non-interactive --force --database-user-name="root" --database-user-password="password" --database-host-name="mariadb" --database-port="3306" --database-name="typo3" --use-existing-database="typo3" --admin-user-name="admin" --admin-password="password" --site-name="EXT:imgdb" --site-setup-type="site"

cat /path/to/dump.sql | $TYPO3CMS_PATH database:import
```

Also you can add `AdditionalConfiguration.php` for disabling Caching or similar purposes. Link your Extensions to `/var/www/html/web/typo3conf/ext/`.

If using host port 80 the website is available under `localhost:80/web` and TYPO3 under `localhost:80/web/typo3`