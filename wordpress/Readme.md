# WordPress server configuration

#### Persistent Logs
 Be sure that the following two folders exist and are placed in the right Apache logs folder.
 
 `/var/logs/wordpress_logs/`
 
 `/var/logs/wp_mysql_logs/`
 
##### Note:
Without the correct linux permissions i.e. the ability to write to the file paths above, the MySQL container will restart continuously.

#### Ensure WordPress wp-content folder path is persistent and temp folder exists

```
## While in Centos server terminal (not wordpress container)
## The temp folder enables uploading files like plugins.zip files and media content. 
sudo mkdir /var/www/html/wp-content 
cd /var/www/html/wp-content
sudo mkdir tmp

## Next access the WordPress container environment
docker exec -it wordpress bash

## To enable file uploads (media/plugins/wpress!)
chown www-data:root wp-content
chown www-data:root wp-content/uploads
```

##### Note:
Plugins will be persistent between newly created WordPress containers, but will have to be activated again after installing WordPress or reconnecting to the saved database if it exists.

## Warning:
The following example reverse proxy IP is set as an .env variable that is passed as an argument to the WordPress Dockerfile, please be sure to update it to your server's IP.

`"RemoteIPTrustedProxy 206.12.91.XXX"`

# Docker Compose run commands

```
## For a standard detached build use and bring the containers online: 
docker-compose up -d --build

## To use a different docker-compose.yml file other than the default, try the following command, this will also add a flag "a_" in front of the build names if desired.
docker-compose -f docker-compose_dockerfile.yml -p="a_" up -d --build

## To stop all the docker containers associated with the docker-compose file.
docker-compose stop

## To stop and remove all the containers/network
docker-compose down
```

## Apache Security Headers in WordPress Dockerfile

Please note that after updating or adding plugins, making CSS style changes or other design changes, it may require updating the Content-Security-Policy header rules to ensure all files are served.
