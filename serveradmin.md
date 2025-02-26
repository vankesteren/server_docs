
- [1. Creating a user account](#1-creating-a-user-account)
  - [1.1. Send the following email to the user](#11-send-the-following-email-to-the-user)
  - [1.2. Create the actual account](#12-create-the-actual-account)
  - [1.3. 2.3 Add the user to the teams page](#13-23-add-the-user-to-the-teams-page)
  - [1.4. Email the user](#14-email-the-user)
- [2. General process monitoring](#2-general-process-monitoring)
- [3. Managing RStudio sessions](#3-managing-rstudio-sessions)
- [4. Fixing R package installation errors](#4-fixing-r-package-installation-errors)
- [5. Temporary folder monitoring](#5-temporary-folder-monitoring)


# 1. Creating a user account

## 1.1. Send the following email to the user

> Dear {user}, 
>
> Thank you for your request for an account on the M&S department compute server. 
> In the URL below is the documentation, please read it carefully and then send me 
> an email to request an account (optionally with your desired username).
>
> https://vankesteren.github.io/server_docs/userdocs
>
> Kind regards,
>
> {your name}


## 1.2. Create the actual account

After receiving the formal request for an account, it is time to create the account. 

```bash
export username=newuser # replace newuser with the username
sudo useradd -m -d /data/$username $username
sudo chown -R $username /data/$username
sudo chmod -R go-rw /data/$username
sudo passwd $username
```

In the last step, create a temporary password and record it.

## 1.3. 2.3 Add the user to the teams page

We also have [a teams page](https://teams.microsoft.com/l/team/19%3A477ed710337644a5b2574c82dbf570cc%40thread.tacv2/conversations?groupId=19e1dc59-adab-480a-b191-027682731102&tenantId=d72758a0-a446-4e0f-a0aa-4bf95a4a10e7) where we stay in contact, ask questions, and generally keep track of who is doing research with the server. Add the user to the page as well.

## 1.4. Email the user 

> Dear {user}, 
>
> An account has been created for you on the department compute server.
>
> Username: {username}
> Temporary password: {password}
>
> Please update your password as soon as possible. 
>
> If you have any questions, do not hesitate to ask! I have added you to the department computer teams page.
>
> Happy computing!
>
> {your name}

# 2. General process monitoring

It's a good idea to check for the usage of the server by different users. Generally, the command `htop` is used to check for server usage, processes open by different users, and more. From there, stuck processes can be killed.

```bash
sudo htop
```

# 3. Managing RStudio sessions

The commandline application `rstudio-server` is used to manage sessions on the RStudio server, and to manage the availability of the server itself.

# 4. Fixing R package installation errors

# 5. Temporary folder monitoring