# Backup procedure and off-site instructions
- Using python-openstackclient command line.

Choose one of the following two options.
### Option A: Using Windows PowerShell and conda environment:

#### Install Anaconda (Miniconda):

Navigate in your browser to:
https://repo.continuum.io/miniconda/

Download either the Windows .exe, MacOS .pkg, or Linux .sh file.
You can select the 'latest' file, or pick an older version you want.
i.e. https://repo.continuum.io/miniconda/Miniconda3-latest-Windows-x86_64.exe
```bash
# Run the Windows Executable 

# Create Miniconda environment and install Python-OpenStackClient dependencies
conda create --name openstack
conda activate openstack
conda install -y python=3
# conda install -y python3-dev
pip install python-openstackclient

# Initiate your terminal with conda
conda init powershell

# You can disable the base conda environment from activating itself whenever you open a fresh terminal
conda config --set auto_activate_base false

# Download provisioning repo if not already done
git clone https://github.com/cioos-atlantic/provisioning.git
cd C:/Users/cioos-data-manager/github/provisioning/openstack

# Activate the conda environment of your choice, i.e. openstack
conda activate openstack

# Activate PowerShell script that parses your OpenStack exported shell profile details
.\Source-OpenRC.ps1 .\sample-project-openrc_v3.sh

# Check that it is working
openstack --version
```

### Option B: Using Bash and conda environment:
Miniconda isn't a necessity, venv can also work. We used Vagrant here to test out the build process.
```bash
# Download provisioning repo if not already done
git clone https://github.com/cioos-atlantic/provisioning.git
cd C:/Users/cioos-data-manager/github/provisioning/openstack/vagrant

# Start vagrant machine (uses vagrantfile)
vagrant up
vagrant ssh

# Download and run Miniconda shell script to install it.
sudo yum -y install wget
wget https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh
chmod 775 Miniconda3-latest-Linux-x86_64.sh
sh Miniconda3-latest-Linux-x86_64.sh
# press Enter
# type 'yes' to accept terms
# Press Enter to confirm location, or specify directory
# Type 'yes' to run 'conda init' command
# Exit to leave terminal session
exit;
vagrant ssh

# You can disable the base conda environment from activating itself whenever you open a fresh terminal
conda config --set auto_activate_base false
exit;
vagrant ssh
conda --version

# Create Miniconda environment and install Python-OpenStackClient dependencies
conda create --name openstack
conda activate openstack

# Note there was a build issue with 'netifaces' in Python version 3.7.3
# TODO: I somehow got it working previously, may have manually installed dependencies?
# Downgrading/choosing lower version of Python is a workaround.
conda install -y python=3.6
# conda install -y python3-dev
pip install python-openstackclient

# Ensure identity file that you exported from Arbutus/OpenStack is available
# If you are copying it into the 'default' vagrant environment now, do the following.
vagrant plugin install vagrant-scp
vagrant scp .\sample-project-openrc_v3.sh default:/home/vagrant/sample-project-openrc_v3.sh

# This ensures variables are available in the current session to be read by OpenStack
source sample-project-openrc_v3.sh

# Activate the conda environment of your choice, i.e. openstack
conda activate openstack
``` 

#### Clone volume and create image of clone to then download off-site backup
```bash
# Enter your OpenStack user credential password (not ssh-key password) on the next command

# List volumes (note the size of volume you want to clone/make a downloadable image you want)
openstack volume list

# Clone an existing detached volume using numeric ID, size must be the same!
openstack volume create --image 7096c1e8-d227-4155-b466-f3e0786b27c5 --size 100 software_test_2019_07_12

# To see the name and ID of each saved image
openstack image list

# To see extra details of the same images
openstack image list --long


# Check logs of an active VM based on ID
openstack console log show --lines 30 3213f694-6ce8-4d23-9c61-c86cf278e786
```
#### Full list of OpenStackClient commands here:
https://docs.openstack.org/python-openstackclient/latest/cli/command-list.html

#### Download off-site backup
 Select and copy the image numeric ID you want to download and replace the ID here
 Note on a high-speed fibre network at 80-100 Mbps download rate:
 - a 15GB image file can take ~30 minutes (50GB volume)
 - a 26GB image file can take ~45-60 minutes (200GB volume)
 
Warning: Over WiFi this download time can quadruple! Not sure how resilient the process is to disconnecting either! Also assuming one has space on an SSD. 

```bash
openstack image save --file .\images\software_test_june_2019_08_02_dwl.qcow2 5ed14ed6-8a1f-4491-a1dd-24b462e87a59
```
