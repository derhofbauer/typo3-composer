# typo3-composer

This repository contains build instructions for a simple TYPO3 Composer Mode Docker image, bundeling the typo3_console extensions.

## Usage

This Container does not ship a database management system; I recommend to use `docker-compose` for this purpose.

You can use this image for programmatically setting up a TYPO3 Environment. Use a shell script for installing TYPO3 and importing a DB Dump:

```bash
$TYPO3CMS_PATH install:setup --non-interactive \
                             --force \
                             --database-user-name="root" \
                             --database-user-password="password" \
                             --database-host-name="mariadb" \
                             --database-port="3306" \
                             --database-name="typo3" \
                             --use-existing-database="typo3" \
                             --admin-user-name="admin" \
                             --admin-password="password" \
                             --site-name="My Demo Site" \
                             --site-setup-type="site"

cat /path/to/dump.sql | $TYPO3CMS_PATH database:import
```

Also you can add `AdditionalConfiguration.php` for disabling Caching or similar purposes. Link your Extensions to `/var/www/html/web/typo3conf/ext/`.

If using host port 80 the website is available under `localhost:80/web` and TYPO3 under `localhost:80/web/typo3`

## Examples

A sample `docker-compose.yml` file could look like this:

```yml
version: '2.0'

services:
  mariadb:
    image: mariadb
    ports:
      - 3306:3306
    environment:
      MYSQL_ROOT_PASSWORD: 'password'
      MYSQL_DATABASE: 'typo3'
      MYSQL_USER: 'typo3'
      MYSQL_PASSWORD: 'password'

  typo3:
    image: derhofbauer/typo3-composer
    volumes:
      - './setup/:/setup'
      - './src/:/src'
      - './setup/data/files:/var/www/html/web/fileadmin/user_upload/testdata'
    ports:
      - '8080:80'
    links:
      - mariadb:db
    depends_on:
      - mariadb
    command: bash -c '/setup/setup.sh && apache2-foreground'
```

The corresponding `setup/setup.sh` file could look like this:

```bash
EXT_KEY='sample-sitepackage'
TYPO3CMS_PATH='vendor/bin/typo3cms'

if [[ ! -f "INSTALLATION_DONE" ]]; then
	chmod 777 . -R
	echo "install:generatepackagestates"
	$TYPO3CMS_PATH install:generatepackagestates
	chmod 777 . -R
	echo "install:fixfolderstructure"
	$TYPO3CMS_PATH install:fixfolderstructure
	chmod 777 . -R

	echo "install:setup"
	$TYPO3CMS_PATH install:setup --non-interactive --force --database-user-name="root" --database-user-password="password" --database-host-name="mariadb" --database-port="3306" --database-name="typo3" --use-existing-database="typo3" --admin-user-name="admin" --admin-password="password" --site-name="Sample Sitepackage" --site-setup-type="site"
	
	if [[ -f "/setup/data/sql/dump.sql" ]]; then
		echo "Database Import"
		cat /setup/data/sql/dump.sql | $TYPO3CMS_PATH database:import
	fi
fi

if [[ ! -f "/var/www/html/web/.htaccess" ]]; then
	cp /var/www/html/vendor/typo3/cms/_.htaccess /var/www/html/web/.htaccess
fi

chmod 777 /var/www -R
chown www-data:www-data -R /var/www

if [[ ! -d web/typo3conf/ext/$EXT_KEY ]]; then
	ln -s /src/ web/typo3conf/ext/$EXT_KEY
fi

$TYPO3CMS_PATH extension:activate --extension-keys=realurl,news,powermail,gridelements,scheduler
$TYPO3CMS_PATH extension:activate $EXT_KEY
```

The `setup/data/sql/dump.sql` file was created using the Typo3 Console extension:

```bash
/var/www/html/vendor/bin/typo3cms database:export --exclude-tables be_sessions,fe_sessions,sys_log > /setup/data/sql/dump.sql
```