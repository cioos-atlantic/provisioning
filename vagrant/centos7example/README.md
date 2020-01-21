# vagrant

We are currently using vagrant's private networking option. This means we can assign the local IP address and access from Chrome/DevTools.
It also means we don't have to specifically forward every port that we may want at a later date for each internal Docker container.
https://www.vagrantup.com/docs/networking/private_network.html

The vagrantfile included here is a basic Centos7 box setup.
The shell instructions which will be included, will automatically provision the Docker and CKAN environment. 

## Note:
````
It might be wise to leave the following command out of the shell script due to how long it takes to run:
sudo docker-compose up -d --build
````

# CKAN

CKAN build time will vary depending on the hardware, but locally it has been about 15-30 minutes depending on if it needs to download the images the first time, or how many Docker caches it can use in subsequent builds.

See the following for more information on build process instructions: https://github.com/cioos-siooc/ckan 