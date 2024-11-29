# Server administration

- [Server administration](#server-administration)
  - [Creating a user account](#creating-a-user-account)
    - [Send the following email to the user](#send-the-following-email-to-the-user)
    - [Create the actual account](#create-the-actual-account)
    - [Email the user](#email-the-user)
  - [Temporary folder monitoring](#temporary-folder-monitoring)
  - [Managing RStudio sessions](#managing-rstudio-sessions)
  - [Fixing R package installation errors](#fixing-r-package-installation-errors)


## Creating a user account

### Send the following email to the user

> Dear {user}, 
>
> Thank you for your request for an account on the M&S department compute server. 
> In the URL below is the documentation, please read it carefully and then send me 
> an email to request an account once you're done!
>
> https://msserver.fss.uu.nl/docs


### Create the actual account
```bash
export username=newuser
sudo useradd -m -d /data/$username $username
sudo chown -R $username /data/$username
sudo chmod -R go-rw /data/$username
sudo passwd $username
```

### Email the user 

## Temporary folder monitoring



## Managing RStudio sessions



## Fixing R package installation errors