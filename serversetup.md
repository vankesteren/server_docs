Here are the steps to follow in order to set up our department compute server

- [1. Initial setup](#1-initial-setup)
  - [1.1. Request a virtual machine](#11-request-a-virtual-machine)
  - [1.2. Mount the data disk](#12-mount-the-data-disk)
  - [1.3. Create admin account](#13-create-admin-account)
  - [1.4. Update packages](#14-update-packages)
  - [1.5. Add admin user management scripts](#15-add-admin-user-management-scripts)
    - [1.5.1. Create a `newuser` program](#151-create-a-newuser-program)
    - [1.5.2. Create a `removeuser` program](#152-create-a-removeuser-program)
    - [1.5.3. Create a `backupuser` program](#153-create-a-backupuser-program)
- [2. Installing R and RStudio server](#2-installing-r-and-rstudio-server)
  - [2.1. Installing R](#21-installing-r)
    - [2.1.1. Installing R through the package manager](#211-installing-r-through-the-package-manager)
    - [2.1.2. Compiling R from source](#212-compiling-r-from-source)
  - [2.2. Configuring R](#22-configuring-r)
  - [2.3. Installing RStudio server](#23-installing-rstudio-server)
- [3. Setting up a reverse proxy](#3-setting-up-a-reverse-proxy)
  - [3.1. Setting up SSL keyfiles](#31-setting-up-ssl-keyfiles)
  - [3.2. Installing nginx](#32-installing-nginx)
  - [3.3. Configuring nginx](#33-configuring-nginx)
- [4. Conclusion](#4-conclusion)



# 1. Initial setup

## 1.1. Request a virtual machine

Ask FSBS IT department to set up the latest LTS ubuntu server machine on their server infrastructure, with ports 22 (SSH), 80 (HTTP), and 443 (HTTPS) open. Request a url for the server, we will assume this is `msserver.fss.uu.nl`. Request a boot disk (`/dev/sda`) of about 100GB and a scratch / data disk (`/dev/sdb`) of about 2TB

> Our contacts at the IT department are Halim Skori and Martijn van Ackooij

## 1.2. Mount the data disk

The data disk will contain the home directories of the users, as well as any data they have.

First, SSH into the server using the provided admin account (assumed to be `labgenius`)

```bash
ssh labgenius@msserver.fss.uu.nl
# now enter the password you received from IT
```

Then, check if the hard disk is there

```bash
sudo fdisk -l
```

Then, follow [this guide](https://askubuntu.com/a/154184) to create an ext4 formatted partition on `/dev/sdb`. Note that you probably need to press G instead of O in the first step and some things may be slightly different.

Then, to mount the new `/dev/sdb1` partition, you have to edit `/etc/fstab` (CAREFULLY), add the following:

```
# sdb1 mounted on data
/dev/sdb1 /data ext4 defaults 0 1
```

## 1.3. Create admin account
Now, create your own admin account (we'll use `erikjan`):

```bash
ssh labgenius@msserver.fss.uu.nl
# now enter the password

sudo useradd -m -d /data/erikjan erikjan
sudo chown -R erikjan /data/erikjan
sudo chmod -R go-rw /data/erikjan
sudo adduser erikjan sudo
sudo passwd erikjan
```

Now log out of the default account and log into your own account.

## 1.4. Update packages

Run the following commands to update everything and install the main required packages for our server.

```bash
# update and upgrade
sudo apt update
sudo apt upgrade

# change default shell to zsh for convenience
sudo apt install zsh
chsh -s /bin/zsh

# run the following if the server was in stalled in minimized mode
sudo unminimize

# install a boatload more software
# some of these are packages, some of these are libraries needed to install R packages
sudo apt install htop gdebi-core make build-essential libcurl4-openssl-dev zlib1g-dev libxml2-dev libfontconfig1-dev libharfbuzz-dev libfribidi-dev libfreetype6-dev libpng-dev libtiff5-dev libjpeg-dev gfortran libblas-dev liblapack-dev libapparmor-dev cmake libudunits2-dev libgdal-dev gdal-bin software-properties-common dirmngr cargo rustc pandoc

# now a reboot may be in order
sudo reboot 
```

After the machine has rebooted, ssh into it again with your account.

## 1.5. Add admin user management scripts

Several scripts can make adminning a bit easier and more user-friendly.

### 1.5.1. Create a `newuser` program

```bash
nano newuser
```

Then, copy-paste [this bash script](newuser.sh) (made with some LLM help)

Then, save this script (`ctrl+X`, then `Y`).

Then, we move this to `/usr/local/bin` and set it to be executable:

```bash
sudo mv newuser /usr/local/bin/newuser
sudo chmod +x /usr/local/bin/newuser
```

Now, we can add users via `newuser username password`.

### 1.5.2. Create a `removeuser` program

```bash
nano removeuser
```

Then, copy-paste [this bash script](removeuser.sh) (made with some LLM help)

Then, save this script (`ctrl+X`, then `Y`).

Then, we move this to `/usr/local/bin` and set it to be executable:

```bash
sudo mv removeuser /usr/local/bin/removeuser
sudo chmod +x /usr/local/bin/removeuser
```

### 1.5.3. Create a `backupuser` program

```bash
nano backupuser
```

Then, copy-paste [this bash script](backupuser.sh) (made with some LLM help)

Then, save this script (`ctrl+X`, then `Y`).

Then, we move this to `/usr/local/bin` and set it to be executable:

```bash
sudo mv backupuser /usr/local/bin/backupuser
sudo chmod +x /usr/local/bin/backupuser
```


# 2. Installing R and RStudio server

## 2.1. Installing R

There are two options for installing R, either via `apt` or by compiling from source. I will show both here, but for the latest server we need to do the latter to enable full use of the 240 cores. To see if this is still needed, keep an eye on [this github issue](https://github.com/rstudio/rstudio/issues/15360) For both methods, you should first set up the official R apt repository source. The below instructions come directly from CRAN [here](https://cran.r-project.org/bin/linux/ubuntu/) in November 2024 and may need to be adjusted for later versions of R.

```bash
# update indices
sudo apt update -qq
# install two helper packages we need
sudo apt install --no-install-recommends software-properties-common dirmngr
# add the signing key (by Michael Rutter) for these repos
# To verify key, run gpg --show-keys /etc/apt/trusted.gpg.d/cran_ubuntu_key.asc 
# Fingerprint: E298A3A825C0D65DFD57CBB651716619E084DAB9
wget -qO- https://cloud.r-project.org/bin/linux/ubuntu/marutter_pubkey.asc | sudo tee -a /etc/apt/trusted.gpg.d/cran_ubuntu_key.asc
# add the R 4.0 repo from CRAN
sudo add-apt-repository "deb https://cloud.r-project.org/bin/linux/ubuntu $(lsb_release -cs)-cran40/"
```


### 2.1.1. Installing R through the package manager
Installing R through the package manager is now easy peasy:

```bash
sudo apt install --no-install-recommends r-base
```

Now there will be an R version in `/usr/bin/R`, and the installation will be at `/usr/lib/R`.

### 2.1.2. Compiling R from source

To use all the cores, we need to create a version of R with a higher-than-default number of simultaneously allowed "connections". For this, we need to download the source code of R itself, then adjust a value in the file `connections.c` and then compile R.

More info on compiling R from source is [here](https://docs.posit.co/resources/install-r-source.html), and more info on this connections thing is [here](https://search.r-project.org/CRAN/refmans/parallelly/html/availableConnections.html#How-to-increase-the-limit).

First, add the R source repository to the apt sources using the built-in [`nano`](https://en.wikipedia.org/wiki/GNU_nano) text editor

```bash
# find the name of the source list
sudo ls /etc/apt/sources.list.d
# for me it's the file with the beautiful name archive_uri-https_cloud_r-project_org_bin_linux_ubuntu-noble.list
sudo nano /etc/apt/sources.list.d/archive_uri-https_cloud_r-project_org_bin_linux_ubuntu-noble.list
```

This file will look something like this:

```bash
deb https://cloud.r-project.org/bin/linux/ubuntu noble-cran40/
# deb-src https://cloud.r-project.org/bin/linux/ubuntu noble-cran40/
```

Remove the comment symbol to make it look like this:

```bash
deb https://cloud.r-project.org/bin/linux/ubuntu noble-cran40/
deb-src https://cloud.r-project.org/bin/linux/ubuntu noble-cran40/
```

To close and save the updated file, press `ctrl+x` to close it and then `y` to save.

Now, we can install build dependencies:

```bash
sudo apt update
sudo apt build-dep r-base
```

Now it's time to download the R source code

```bash
# Update the version in the lines below
# download the sources
curl -O https://cran.r-project.org.com/src/base/R-4/R-4.4.2.tar.gz
# uncompress the source folder
tar -xzvf R-4.4.2.tar.gz
# move to that folder
cd R-4.4.2
```

Then, we are actually going to edit the source code:

```bash
sudo nano src/main/connections.c
```

Now scroll down to the line containing

```c
static int NCONNECTIONS = 128; /* need one per cluster node */
```

and change the value 128 to something reasonable like 512. Then `ctrl+x`, `y` and we're ready to build R!

```bash
# again change the R version to the correct one
./configure \
  --prefix=/opt/R/4.4.2 \
  --enable-R-shlib \
  --enable-memory-profiling \
  --with-blas \
  --with-lapack

make -j 24
sudo make install
```

Done! Now you have R installed under `/opt/R/4.4.2`. You should make links to `/usr/bin/` so you can run R by just typing `R` in the console, and so RStudio Server can find R:

```bash
sudo ln -s /opt/R/4.4.2/bin/R /usr/bin/R
sudo ln -s /opt/R/4.4.2/bin/Rscript /usr/bin/Rscript
```
## 2.2. Configuring R

Install basic packages in the main library that everyone gets to use:
```bash
sudo R
install.packages(c("tidyverse", "devtools"), Ncpus = 24)
q("no")
```
If there are issues with this, refer to the [relevant server administration page](./serveradmin#4-fixing-r-package-installation-errors).

## 2.3. Installing RStudio server

RStudio server is installed basically as standard, following the main installation documentation [here](https://posit.co/download/rstudio-server/)


```bash
# this will change based on the version of RStudio and the version of the server
wget https://download2.rstudio.org/server/jammy/amd64/rstudio-server-2024.12.1-563-amd64.deb
sudo gdebi rstudio-server-2024.12.1-563-amd64.deb
```

Then, the RStudio server will need to be configured as well.

```bash
sudo nano /etc/rstudio/rserver.conf
```

This file should look like this:

```
# Server Configuration File
www-port=8787
```

Then `ctrl+x`, `y` and move on to the next configuration file:

```bash
sudo nano /etc/rstudio/rsession.conf
```

Which should look like this:

```
# R Session Configuration File
session-timeout-minutes=0

# library path
r-libs-user=~/R/packages
```

Now, we should stop and restart RStudio server and it will be ready!
```bash
sudo rstudio-server stop
sudo rstudio-server verify-installation
sudo rstudio-server start
```

> NB: the server will listen on port 8787, which is not accessible from outside the server itself. We need to set up a reverse proxy to make the RStudio service available. This is done in the next section.

# 3. Setting up a reverse proxy

We will use `nginx` to set up a reverse proxy and to enable secure connection over `https` to the server (over and above the required vpn security). This requires installing `nginx` and then configuring it using its native configuration file language for our specific purpose.

## 3.1. Setting up SSL keyfiles

We want to make the server available through a secure connection over https. Therefore, we need to create SSL cryptographic that our nginx server can refer to. Those files will need to be in the following locations:

```
/etc/cert/msserver_fss_uu_nl.pem
/etc/cert/msserver.fss.uu.key
```

First, we will create the key file on the server. For this, run the following code: 

```bash
openssl req -new -newkey rsa:2048 -nodes -keyout msserver.fss.uu.key -out msserver.fss.uu.csr
```

You will be asked interactively to fill out some additional information for the certificate signing authority to refer to. Fill out those. 

Move the key file to the required location:
```bash
sudo mkdir /etc/cert
sudo mv msserver.fss.uu.key /etc/cert/msserver.fss.uu.key
```

Then send the `.csr` file to the IT department so they can have the key signed. That will yield the `.pem` file, which should be uploaded and put in the correct location (`/etc/cert/msserver_fss_uu_nl.pem`).


## 3.2. Installing nginx

Installing nginx is as easy as downloading it from the ubuntu repo:

```bash
sudo apt install nginx
```

This already enables the webserver which listens for incoming traffic and can then do stuff to serve different things. 

## 3.3. Configuring nginx

To make nginx route traffic from http (port 80) and https (port 443) to Rstudio server (port 8787), we need to configure it. Configuration of nginx happens in configuration files in the folder `/etc/nginx/sites-available/` and `/etc/nginx/sites-enabled/`.

First, create and edit a configuration file called `reverse_proxy.conf` in the sites-available directory: 

```bash
sudo nano /etc/nginx/sites-available/reverse_proxy.conf
```

Then, ensure that the content of this file is the same as the file here: [reverse_proxy.conf](./reverse_proxy.conf). A few things happen there:

- Reroute any http traffic to https
- Reroute msserver.fss.uu.nl/docs to this documentation website
- Set up SSL (secure traffic over https) with certificates
- Turn off the access log
- Forward any website traffic to RStudio server

Save the file (`ctrl+x` and then `y`) and then enable the site by creating a link to it in the sites-enabled directory:

```bash
sudo ln -s /etc/nginx/sites-available/reverse_proxy.conf /etc/nginx/sites-enabled/reverse_proxy.conf
```

RStudio server should now be available at https://msserver.fss.uu.nl from the university network or via VPN!

# 4. Conclusion

We have now set up the basic form of the department compute server. To operate the server, users need to receive an account, additional software needs to be installed and kept up-to-date, and the server needs to be generally maintained. Those ongoing activities are documented in the [server administration guide](./serveradmin).