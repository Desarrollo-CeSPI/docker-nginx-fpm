# NGINX docker image that integrates with php-fpm

This image is for easily integrate nginx with php-fpm images. You should export
php project's static files in a volume that shall be read by this image. Then
some variables must be set for integration:

* **ROOT:** nginx document root. Where assets are published
* **PHP\_INDEX\_FILE:** it always is index.php, but you can set it with your php
  script file name
* **PHP\_FILES:** which php script files will be allowed to execute
* **PHP\_UPSTREAM:** upstream container name and port to access FPM service
* **PHP\_ROOT:** document root for php-fpm script. It can be different than
* **NGINX_GLOBAL_CONFIG**: allows you to add custom nginx config at http {}
  block level
  document_root

## Example usage

The following is a sample symfony 1.4 application

```yaml
version: '2'
volumes:
  app-www:
  app-db:
services:
  app:
  # this image is created from php:fpm
  # application code will be inside /app
  # symfony's app/web directory will be copied to /web using cp -arL so
  # symlinks are removed and files can be accede by nginx:
  #   chown www-data:www-data /web && rm -fr /web/* && cp -arL /app/web/* /web/
    image: my-fpm-based-image
      DB_NAME: my_app
      DB_HOST: db
      DB_USERNAME: root
      DB_PASSWORD: root_pass
    volumes:
      - app-www:/web
  nginx:
    image: cespi/nginx-fpm:1.13
    environment:
      ROOT: /web
      PHP_INDEX_FILE: index.php
      PHP_FILES: index
      PHP_UPSTREAM: app:9000
      PHP_ROOT: /app/web
      NGINX_GLOBAL_CONFIG: 'set_real_ip_from 10.42.0.0/16;'
    ports:
      - "8080:80"
    volumes:
      - app-www:/web
  db:
    image: mysql:5.5
    environment: 
      MYSQL_DATABASE: my_ap
      MYSQL_ROOT_PASSWORD: root_pass
    volumes:
      - app-db:/var/lib/mysql
```

Integration key is the commented section of app image. It's image entrypoint
will write php.ini and copy `/app/web/*` files into `/web` without symlinks to
be used by nginx

