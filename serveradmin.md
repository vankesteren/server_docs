# 1. Server administration

- [1. Server administration](#1-server-administration)
- [2. Creating a user account](#2-creating-a-user-account)
  - [2.1. Send the following email to the user](#21-send-the-following-email-to-the-user)
  - [2.2. Create the actual account](#22-create-the-actual-account)
  - [2.3. Email the user](#23-email-the-user)
- [3. Managing RStudio sessions](#3-managing-rstudio-sessions)
- [4. Fixing R package installation errors](#4-fixing-r-package-installation-errors)
- [5. Temporary folder monitoring](#5-temporary-folder-monitoring)


# 2. Creating a user account

## 2.1. Send the following email to the user

> Dear {user}, 
>
> Thank you for your request for an account on the M&S department compute server. 
> In the URL below is the documentation, please read it carefully and then send me 
> an email to request an account once you're done!
>
> https://msserver.fss.uu.nl/docs


## 2.2. Create the actual account
```bash
export username=newuser
sudo useradd -m -d /data/$username $username
sudo chown -R $username /data/$username
sudo chmod -R go-rw /data/$username
sudo passwd $username
```

## 2.3. Email the user 

# 3. Managing RStudio sessions

# 4. Fixing R package installation errors

# 5. Temporary folder monitoring