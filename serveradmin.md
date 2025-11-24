
- [1. Creating a user account](#1-creating-a-user-account)
  - [1.1. Send the following email to the user](#11-send-the-following-email-to-the-user)
  - [1.2. Create the actual account](#12-create-the-actual-account)
  - [1.3. Add the user to the teams page](#13-add-the-user-to-the-teams-page)
  - [1.4. Email the user](#14-email-the-user)
  - [1.5. Add/remove admin rights](#15-addremove-admin-rights)
- [2. Deleting a user account](#2-deleting-a-user-account)
- [3. Server monitoring](#3-server-monitoring)
  - [3.1. Process monitoring](#31-process-monitoring)
  - [3.2. Main storage monitoring](#32-main-storage-monitoring)
  - [3.3. Temporary folder monitoring](#33-temporary-folder-monitoring)
- [4. Managing RStudio sessions](#4-managing-rstudio-sessions)
- [5. Fixing R package installation errors](#5-fixing-r-package-installation-errors)
- [6. Updating R](#6-updating-r)


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

After receiving the formal request for an account, it is time to create the account. We have some nice user-friendly scripts installed on the system to do this (`adm` stands for admin):

```bash
sudo adm user add <username> <password> <emailaddress>
```

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

# 2. Deleting a user account
Deleting a user account happens in two steps. First, we may (optionally) back up the user's home directory

```bash
sudo backupuser <username>
```
then you can use `scp` to download the compressed archive `username.tar.gz` in the working directory. Then, delete the file because it is probably huge.

Then, we will properly remove the user

```bash
sudo removeuser <username>
```

This asks for confirmation before doing anything, so don't worry too much :)


# 3. Server monitoring

It's a good idea to regularly check for the usage of the server by different users. 

## 3.1. Process monitoring
Generally, the command `htop` is used to check for server usage, processes open by different users, and more. From there, stuck processes can be killed.

```bash
sudo htop
```

## 3.2. Main storage monitoring
To ensure the `/data` disk is not filled up, you can check the overall usage and per-user usage using the following commands:

```bash
df -h /data
sudo du -hs /data/*
```

If any user is using outrageous amounts of data, tell them to download it via the "Backing up your data" section in the [user docs](./userdocs) and then delete it.


## 3.3. Temporary folder monitoring

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

# 4. Managing RStudio sessions

The commandline application `rstudio-server` is used to manage sessions on the RStudio server, and to manage the availability of the server itself.

# 5. Fixing R package installation errors


# 6. Updating R
Update the R version in the [userdocs](./userdocs).