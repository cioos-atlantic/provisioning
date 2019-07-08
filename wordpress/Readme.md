# WordPress server configuration

#### Persistent Logs File paths
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
Currently the following IP is hardcoded in the WordPress Dockerfile
`"RemoteIPTrustedProxy 206.12.91.177"`

