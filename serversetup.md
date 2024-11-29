# 1. Configuring the server

Here are the steps to follow in order to set up our department compute server

- [1. Configuring the server](#1-configuring-the-server)
- [2. Initial setup](#2-initial-setup)
  - [2.1. Request a virtual machine](#21-request-a-virtual-machine)
  - [2.2. Mount the data disk](#22-mount-the-data-disk)
  - [2.3. Create admin account](#23-create-admin-account)
  - [2.4. Update packages](#24-update-packages)
- [3. Installing R and RStudio server](#3-installing-r-and-rstudio-server)
  - [3.1. Installing R](#31-installing-r)
    - [3.1.1. Installing R through the package manager](#311-installing-r-through-the-package-manager)
    - [3.1.2. Compiling R from source](#312-compiling-r-from-source)
  - [3.2. Configuring R](#32-configuring-r)
  - [3.3. Installing RStudio server](#33-installing-rstudio-server)



# 2. Initial setup

## 2.1. Request a virtual machine

Ask FSBS IT department to set up the latest LTS ubuntu server machine on their server infrastructure, with ports 22 (SSH), 80 (HTTP), and 443 (HTTPS) open. Request a url for the server, we will assume this is `msserver.fss.uu.nl`. Request a boot disk (`/dev/sda`) of about 100GB and a scratch / data disk (`/dev/sdb`) of about 2TB

> Our contacts at the IT department are Halim Skori and Martijn van Ackooij

## 2.2. Mount the data disk

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

## 2.3. Create admin account
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

## 2.4. Update packages

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
sudo apt install htop gdebi-core make build-essential libcurl4-openssl-dev zlib1g-dev libxml2-dev libfontconfig1-dev libharfbuzz-dev libfribidi-dev libfreetype6-dev libpng-dev libtiff5-dev libjpeg-dev gfortran libblas-dev liblapack-dev cmake libudunits2-dev software-properties-common dirmngr cargo rustc

# now a reboot may be in order
sudo reboot 
```

After the machine has rebooted, ssh into it again with your account.

# 3. Installing R and RStudio server

## 3.1. Installing R

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


### 3.1.1. Installing R through the package manager
Installing R through the package manager is now easy peasy:

```bash
sudo apt install --no-install-recommends r-base
```

Now there will be an R version in `/usr/bin/R`, and the installation will be at `/usr/lib/R`.

### 3.1.2. Compiling R from source

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
## 3.2. Configuring R

Install basic packages in the main library that everyone gets to use:
```bash
sudo R
install.packages(c("tidyverse", "devtools"), Ncpus = 24)
q("no")
```
If there are issues with this, refer to the [relevant server administration page](./serveradmin#4-fixing-r-package-installation-errors).

## 3.3. Installing RStudio server

