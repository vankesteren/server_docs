# Server setup documentation

Here are the steps to follow in order to set up our department compute server

# Initial setup

## Request a virtual machine

Ask FSBS IT department to set up the latest LTS ubuntu server machine on their server infrastructure, with ports 22 (SSH), 80 (HTTP), and 443 (HTTPS) open. Request a url for the server, we will assume this is `msserver.fss.uu.nl`. 

> Our contacts at the it department are Halim Skori and Martijn van Ackooij

## Create admin account
SSH into the server using the provided admin account (assumed to be `labgenius`), and create your own admin account (we'll use `erikjan`):

```bash
ssh labgenius@msserver.fss.uu.nl
# now enter the password

sudo useradd -m -d /mnt/$username $username
sudo adduser erikjan sudo
sudo chown -R $username /mnt/$username
sudo chmod -R go-rw /mnt/$username
sudo passwd $username
```

Now log out of the default account and log into your own accounts.

## Update packages

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
sudo apt install htop gdebi-core make build-essential libcurl4-openssl-dev zlib1g-dev libxml2-dev libfontconfig1-dev libharfbuzz-dev libfribidi-dev libfreetype6-dev libpng-dev libtiff5-dev libjpeg-dev gfortran libblas-dev liblapack-dev cmake libudunits2-dev software-properties-common dirmngr

# now a reboot may be in order
sudo reboot 
```

After the machine has rebooted, ssh into it again with your account.

# Installing R and RStudio server

## Installing R

There are two options for installing R, either via `apt` or by compiling from source. I will show both here, but for the latest server we need to do the latter to enable full use of the 240 cores. For both methods, you should first set up the official R apt repository source. The below instructions come directly from CRAN [here](https://cran.r-project.org/bin/linux/ubuntu/) in November 2024 and may need to be adjusted for later versions of R.

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


### Installing R through the package manager
Installing R through the package manager is now easy peasy:

```bash
sudo apt install --no-install-recommends r-base
```

Now there will be an R version in `/usr/bin/R`, and the installation will be at `/usr/lib/R`.

### Compiling R from source

To use all the cores, we need to create a version of R with a higher-than-default number of simultaneously allowed "connections". For this, we need to download the source code of R itself, then adjust a value in the file `connections.c` and then compile R.

More info on compiling R from source is [here](https://docs.posit.co/resources/install-r-source.html), and more info on this connections thing is [here]()

First, add the R source repository to the apt sources using the [`nano`]() text editor

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

Now look up the line containing
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

Done! Now you have R installed under /opt/R/4.4.2. You can make links to /usr/bin/:

```bash
sudo ln -s /opt/R/4.4.2/bin/R /usr/bin/R
sudo ln -s /opt/R/4.4.2/bin/Rscript /usr/bin/Rscript
```
## Configuring R

Install basic packages in the main library that everyone gets to use:
```bash
sudo R
install.packages(c("tidyverse", "devtools"), Ncpus = 24)
q("no")
```
If there are issues with this, refer to the [relevant server administration page](./serveradmin#fixing-r-package-installation-errors).

## Installing RStudio server

