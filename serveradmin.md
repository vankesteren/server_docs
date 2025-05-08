
- [1. Creating a user account](#1-creating-a-user-account)
  - [1.1. Send the following email to the user](#11-send-the-following-email-to-the-user)
  - [1.2. Create the actual account](#12-create-the-actual-account)
  - [1.3. Add the user to the teams page](#13-add-the-user-to-the-teams-page)
  - [1.4. Email the user](#14-email-the-user)
  - [1.5. Add/remove admin rights](#15-addremove-admin-rights)
- [2. Server monitoring](#2-server-monitoring)
  - [2.1. Process monitoring](#21-process-monitoring)
  - [2.2. Main storage monitoring](#22-main-storage-monitoring)
  - [2.3. Temporary folder monitoring](#23-temporary-folder-monitoring)
- [3. Managing RStudio sessions](#3-managing-rstudio-sessions)
- [4. Fixing R package installation errors](#4-fixing-r-package-installation-errors)
- [5. Updating R](#5-updating-r)


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
sudo chown -R $username /data/$username # set user to owner of 
sudo chmod -R go-rwx /data/$username # remove read write and execute permissions for anyone but owner
sudo passwd $username
```

In the last step, create a temporary password and record it.

## 1.3. Add the user to the teams page

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

## 1.5. Add/remove admin rights

If a user should be an admin, run the following

```sh
sudo usermod -aG sudo <username>
```

Also:
- add them to the contact information in the userdocs
- make them an owner on the microsoft teams page 

If you want to remove a sudoer, run 

```sh
# (this is untested, please check that it works)
sudo deluser <username> sudo
```

Also:
- remove them from the userdocs contact table
- make them a member (not an owner) of the microsoft teams page

# 2. Server monitoring

It's a good idea to regularly check for the usage of the server by different users. 

## 2.1. Process monitoring
Generally, the command `htop` is used to check for server usage, processes open by different users, and more. From there, stuck processes can be killed.

```bash
sudo htop
```

## 2.2. Main storage monitoring
To ensure the `/data` disk is not filled up, you can check the overall usage and per-user usage using the following commands:

```bash
df -h /data
sudo du -hs /data/*
```

If any user is using outrageous amounts of data, tell them to download it via the "Backing up your data" section in the [user docs](./userdocs) and then delete it.


## 2.3. Temporary folder monitoring

Some packages in R create a lot of data in the `/tmp` directory. Since this is on the (small) main boot disk, this can crash out RStudio server ("white screen of death"). To check available space, run

```sh
sudo df -h /tmp
```

To check for particularly big Rtmp folders:

```sh
sudo du -hs /tmp/Rtmp*
```

Then delete a folder using
```sh
# example
sudo rm -r /tmp/Rtmp7jVKku
```

(Nuclear option, do not run while users are running stuff):
```sh
sudo find /tmp/Rtmp* -maxdepth 0 | xargs sudo rm -r
```

Known use-cases that have this problem are:

- `cmdstanr` or `brms`: these save posterior samples as `csv` files in an R temp directory. Mitigation through creating a `~/tmp` directory, and then setting `options(cmdstanr_output_dir = "~/tmp")` and using `backend = "cmdstanr"`. Within the simulation, this directory should be emptied regularly.
- 

# 3. Managing RStudio sessions

The commandline application `rstudio-server` is used to manage sessions on the RStudio server, and to manage the availability of the server itself.

# 4. Fixing R package installation errors


# 5. Updating R
Update the R version in the [userdocs](./userdocs).