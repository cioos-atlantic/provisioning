# ERDDAP server configuration

#### Download ERDDAP gold standard repository and run container.

```
## While in Centos server terminal

git pull https://github.com/ioos/erddap-gold-standard.git
cd erddap-gold-standard/

## The following run command mounts several volumes to the erddap-gold-standard sub-folders.
## This means our data will be persistent even if the container is removed/recreated.
sudo docker run -d -p 8080:8080 --restart=unless-stopped --name erddap 
-v $(pwd)/erddap/conf/setenv.sh:/usr/local/tomcat/bin/setenv.sh 
-v $(pwd)/erddap/conf/robots.txt:/usr/local/tomcat/webapps/ROOT/robots.txt 
-v $(pwd)/erddap/content:/usr/local/tomcat/content/erddap 
-v $(pwd)/erddap/data:/erddapData 
-v $(pwd)/datasets:/datasets 
-v /tmp/:/usr/local/tomcat/temp/ axiom/docker-erddap:1.82
```

##### Note:
The data will take a few minutes at the least to populate when you rebuild the container as it processes each dataset.
