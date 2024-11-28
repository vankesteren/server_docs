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
sudo useradd -m -d /mnt/$username $username
sudo chown -R $username /mnt/$username
sudo chmod -R go-rw /mnt/$username
sudo passwd $username
```

### Email the user 

## Temporary folder monitoring



## Managing RStudio sessions



## Fixing R package installation errors