# ERDDAP server configuration

See details here:
https://github.com/ioos/erddap-gold-standard

#### Download ERDDAP gold standard repository and run container.

```bash
## While in Centos server terminal

git clone https://github.com/ioos/erddap-gold-standard.git
cd erddap-gold-standard/

## The following run command mounts several volumes to the erddap-gold-standard sub-folders.
## This means our data will be persistent even if the container is removed/recreated.

# It won't run over multiple lines, the following is to make it easier to read
# sudo docker run -d -p 8080:8080 --restart=unless-stopped --name erddap \
# -v $(pwd)/erddap/conf/setenv.sh:/usr/local/tomcat/bin/setenv.sh \
# -v $(pwd)/erddap/conf/robots.txt:/usr/local/tomcat/webapps/ROOT/robots.txt \ 
# -v $(pwd)/erddap/content:/usr/local/tomcat/content/erddap \
# -v $(pwd)/erddap/data:/erddapData \
# -v $(pwd)/datasets:/datasets \
# -v /tmp/:/usr/local/tomcat/temp/ axiom/docker-erddap:1.82

# Run this command to build the container
sudo docker run -d -p 8080:8080 --restart=unless-stopped --name erddap -v $(pwd)/erddap/conf/setenv.sh:/usr/local/tomcat/bin/setenv.sh -v $(pwd)/erddap/conf/robots.txt:/usr/local/tomcat/webapps/ROOT/robots.txt -v $(pwd)/erddap/content:/usr/local/tomcat/content/erddap -v $(pwd)/erddap/data:/erddapData -v $(pwd)/datasets:/datasets -v /tmp/:/usr/local/tomcat/temp/ axiom/docker-erddap:1.82
```

##### Note:
The data will take a few minutes at the least to populate when you rebuild the container as it processes each dataset.
