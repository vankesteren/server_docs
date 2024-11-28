# Server setup documentation



## Basic setup

Ask FSBS IT department to set up the latest LTS ubuntu server machine on their server infrastructure, with ports 22 (SSH), 80 (HTTP), and 443 (HTTPS) open. Request a url for the server, we will assume this is `msserver.fss.uu.nl`. 

> contact info for the it department: Halim Skori ()

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