# Migration Instructions: CKAN

This documentation includes a brief summary of commands to migrate CKAN from prod to dev or other route. 
Modified from the following source:
https://docs.ckan.org/en/2.8/maintaining/installing/install-from-docker-compose.html#migrate-data

##### Transfer resources files
Rsync CKAN storage files as root
```bash
export VOL_CKAN_STORAGE=`sudo docker volume inspect docker_ckan_storage | jq -r -c '.[] | .Mountpoint'`
echo $VOL_CKAN_STORAGE
# outputs /var/lib/docker/volumes/docker_ckan_storage/_data
sudo rsync -Pavvr -e "ssh -X -i /home/username/.ssh/centos_2_rsa" --rsync-path="sudo rsync" username@192.168.153.XXX:/var/lib/docker/volumes/docker_ckan_storage/_data /var/lib/docker/volumes/docker_ckan_storage/_data
```
##### Transfer users
Export Postgres user table (with hashed passwords) and copy.
```bash
# Export user table from source CKAN (206.XXX.XXX.XXX)
# pg_dump -h CKAN_DBHOST -P CKAN_DBPORT -U CKAN_DBUSER -a -O -t user -f user.sql ckan_default
docker exec db pg_dump -h 127.0.0.1 -p 5432 -U ckan -a -O -t user > /home/username/user.sql
# Copy user table into destination CKAN (192.168.153.XXX)
scp -i ~/.ssh/centos_2_rsa username@192.168.153.XXX:/home/username/user.sql /home/username/
```
Import into other VM's database.
```bash
docker cp /home/username/user.sql db:/
docker exec -it db bash
chown root:root user.sql
chmod 755 user.sql

# Enter Postgres ckan database password when prompted
psql -U ckan -h db -f user.sql
```
##### Export/upload organizations, datasets and groups from CKAN database
Setup virtual environment for ckanapi by installing Anaconda (Miniconda3). 

<strong>Note:</strong> Will need to repeat this environment setup in both/all servers.

https://repo.continuum.io/miniconda/
```bash
# If wget not already installed, run the following:
sudo yum -y install wget
cd /home/username/
wget https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh
sudo chmod 755 Miniconda3-latest-Linux-x86_64.sh
sh Miniconda3-latest-Linux-x86_64.sh
exit;

# ssh back in to server (to enable base env temporarily)
conda config --set auto_activate_base false
exit;

# ssh back in to server (to disable auto activation)
# Created conda environment to make a clean install
conda create --name ckanapi

# Update conda and activate new environment
conda update -n base -c defaults conda
conda activate ckanapi
conda install pip
pip install ckanapi
```
Using ckanapi Python package, dump the datasets, organizations and groups tables, from source VM.

https://github.com/ckan/ckanapi#bulk-dumping-and-loading-operations
```bash
ckanapi dump datasets --all -O datasets.jsonl.gz -z -p 3 -r https://apache_user:password@cioosatlantic.ca/ckan
ckanapi dump groups --all -O groups.jsonl.gz -z -p 3 -r https://apache_user:password@cioosatlantic.ca/ckan
ckanapi dump organizations --all -O orgs.jsonl.gz -z -p 3 -r https://apache_user:password@cioosatlantic.ca/ckan
```

From destination VM, use ckanapi to import the files, organizations has to be imported before datasets!
```bash
# Copy files over
scp -i ~/.ssh/centos_2_rsa username@192.168.153.XXX:/home/username/datasets.jsonl.gz /home/username/
scp -i ~/.ssh/centos_2_rsa username@192.168.153.XXX:/home/username/groups.jsonl.gz /home/username/
scp -i ~/.ssh/centos_2_rsa username@192.168.153.XXX:/home/username/orgs.jsonl.gz /home/username/

ckanapi load organizations -I orgs.jsonl.gz -z -p 3 -r http://localhost:5000 -a API_KEY
ckanapi load datasets -I datasets.jsonl.gz -z -p 3 -r http://localhost:5000 -a API_KEY
ckanapi load groups -I groups.jsonl.gz -z -p 3 -r http://localhost:5000 -a API_KEY
```