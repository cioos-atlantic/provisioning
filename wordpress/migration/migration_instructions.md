# Migration Instructions: WordPress

This documentation includes a brief summary of commands to migrate WordPress from prod to dev or other route. 

##### Transfer WordPress files
Rsync the WordPress wp-content folder, this includes media files, plugins, themes and so on. 
i.e. everything that gives the WordPress Docker container it's "state". 
The one thing it doesn't include is the database.
```bash
# Login to destination server. Files are rsync'd from source IP to local folder as root.
sudo rsync -azhlv -e "ssh -X -i /home/username/.ssh/centos_2_rsa" --rsync-path="sudo rsync" username@192.168.153.XXX:/var/www/html/wp-content/ /var/www/html/wp-content/
```
##### Export/upload WordPress database
Find more wp-cli commands here:

https://developer.wordpress.org/cli/commands/
```bash
# Login to source server.
cd /var/www/html/wp-content/

# Details on why skipping lock-tables is useful on MySQL databases using Innodb engine on all the tables
# https://serversforhackers.com/c/mysqldump-with-modern-mysql
# Great list of extra fields that can be passed into wp-cli for mysqldump
# https://dev.mysql.com/doc/refman/5.7/en/mysqldump.html#mysqldump-option-summary
sudo docker run -it --rm --volumes-from wordpress --user xfs:root --network wordpress_default wordpress:cli db export --single-transaction --skip-lock-tables ./wp-content/dev_wp_db_2019_08_23.sql

# Login to destination server.
scp -i ~/.ssh/centos_2_rsa username@192.168.153.XXX:/var/www/html/wp-content/dev_wp_db_2019_08_14.sql /var/www/html/wp-content/
sudo docker run -it --rm --volumes-from wordpress --user xfs:root --network wordpress_default wordpress:cli db import ./wp-content/dev_wp_db_2019_08_14.sql

# Search for existing references to URL or IP in database, make sure to check it's format and context.
sudo docker run -it --rm --volumes-from wordpress --user xfs:root --network wordpress_default wordpress:cli db search "https://dev.cioosatlantic.ca" --stats

# Replace the string you searched for with the destination VM's URL or IP.
sudo docker run -it --rm --volumes-from wordpress --user xfs:root --network wordpress_default wordpress:cli search-replace "https://dev.cioosatlantic.ca" "http://localhost:9500"  --all-tables
cd provisioning/wordpress/
# To see if the changes properly updated in the specific VM and IP you are implementing this on.
sudo docker-compose restart
```
##### Fixing potential temporary styling errors.
Note: When reviewing the content visually to make sure the database and files copied over correctly, 
if there are major CSS styling elements missing, Content Security Policy (CSP) might be blocking 
content at the new address, this can be temporarily disabled in the WordPress docker container.
```bash
docker exec -it wordpress bash
# Can comment out the last few lines that were added using the provisioning/wordpress/wordpress/Dockerfile
nano /etc/apache2/conf-enabled/security.conf
```