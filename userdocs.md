# 1. M&S department computer user documentation

This is the documentation for using the Methods & Statistics department compute server.

- [1. M\&S department computer user documentation](#1-ms-department-computer-user-documentation)
- [2. Connecting to the server](#2-connecting-to-the-server)
- [3. User account](#3-user-account)
  - [3.1. Updating the password](#31-updating-the-password)
- [4. Using the server](#4-using-the-server)
  - [4.1. Server specifications](#41-server-specifications)
  - [4.2. Data storage](#42-data-storage)
    - [4.2.1. Backing up your data](#421-backing-up-your-data)
  - [4.3. R sessions](#43-r-sessions)
  - [4.4. R packages](#44-r-packages)
  - [4.5. Parallel processing](#45-parallel-processing)
  - [4.6. Additional software](#46-additional-software)
    - [4.6.1. How to run additional software with a GUI](#461-how-to-run-additional-software-with-a-gui)
    - [4.6.2. Matlab](#462-matlab)
  - [4.7. GPU](#47-gpu)


# 2. Connecting to the server
Connecting to the server is only available in two ways:
- from our department with an ethernet connection.
- via Utrecht University [vpn](https://vpn.uu.nl) from anywhere. 

To connect, type in your browser the following URL: [msserver.fss.uu.nl](http://msserver.fss.uu.nl). You wil be greeted with an `RStudio` login window. This works best in google chrome or mozilla firefox.

# 3. User account
To access the server, you need a login / user account, which is available on request. A user account needs to be manually created for you. Send an email to the admin (Erik-Jan) for this, _with an explanation of why you want to use the computer_. You will receive a default password which you can change when you first log in.

## 3.1. Updating the password
Login to the server, open a terminal within the `rstudio` browser window (`shift + alt + R`), type `passwd <your-user-name>` (for example `passwd erikjan`) and follow the prompts.

# 4. Using the server
To use the server, abide by these rules:

1. Please read the below carefully. 
2. If you aren't sure about something, read again and then _ask_ before doing.
3. If you misuse the computer, your account will be suspended.

Multiple users can connect to the server at the same time. If you are preparing a script, you can always login to the server. If you want to run a large simulation, please reserve time for this on the [Google sheet schedule](https://docs.google.com/spreadsheets/d/1WcUzKStfb5MK4Rgh5jcyNRwyPfOQJA-CjigHBUd6QZk/edit?usp=sharing).

## 4.1. Server specifications
```
CPU     :  2 x Intel(R) Xeon(R) CPU E5-2650 v4 @ 2.20GHz
Threads :  48
Memory  :  64GB
GPU     :  Nvidia GTX 1080Ti 
Storage :  Main disk : 2TB    WDC WD20EFRX-68EUZN0  /data
           OS disk   : 120GB  INTEL SSDSC2KW120H6   /
```

## 4.2. Data storage
Please exclusively use your home directory (`/data/<your-user-name>/`, or alternatively `~/`), which is on the 2TB main harddrive. No other user can see your files there.

_Example_
```r
my_big_matrix <- matrix(0, 1e4, 1e4)
saveRDS(my_big_matrix, "~/bigfile.rds")
```

### 4.2.1. Backing up your data

> You are responsible for archiving your data. The server is not backed up in any way and we provide no guarantees. Consider your home directory as temporary/scratch storage.

After running simulations, it is wise to archive your results somewhere you can access them in case the hard drive of the server breaks. You can do this from the RStudio server by selecting "download" in the files tab. 

It can also be done using the `scp` command from your own computer via the terminal (if you have it installed).

_Example_

To copy the entire folder `simulation_folder` to the local backup folder `local_backup`, the user `testuser` can run the following command:

```bash
scp -rC testuser@mscomputer.fss.uu.nl:~/simulation_folder local_backup
```

<details>
<summary>Note for admin</summary>
Checking storage space can be done as follows: 

```bash
df -h /data
sudo du -hs /data/*
```

SMART tests for the hard drive (`/dev/sda`) should be run every now and then using `smartctl`. Check if the `RAW_VALUE` column shows `Reallocated_Sector_Ct` and such. The MTBF of the hard disk is 1 million hours, so this should be fine for a while.
```bash
sudo smartctl -a /dev/sda
sudo smartctl -t short /dev/sda
sudo smartctl -a /dev/sda
```
</details>

## 4.3. R sessions
When you log in, you start an `R` session. It will remain open until you stop it (red button in the top right corner). Please close your `R` session when you are done. The R version is the following:

```
R version 4.2.3 (2023-03-15) -- "Shortstop Beagle"
Copyright (C) 2023 The R Foundation for Statistical Computing
Platform: x86_64-pc-linux-gnu (64-bit)
```

## 4.4. R packages
Everyone gets their own personal `R` package repository automatically in `~/R/packages`. There should not be any interference with package versions.

If `R` warns you that you need a library for installing packages (e.g. `libssl-dev`), please contact the admin to install it for you. This is what such a warning may look like upon trying to install a package:

```
checking mpfr.h usability... no
checking mpfr.h presence... no
checking for mpfr.h... no
configure: error: Header file mpfr.h not found; maybe use --with-mpfr-include=INCLUDE_PATH
```

In addition, global packages are installed for everyone to use by default. If you want to use a newer version of a preinstalled package, simply install it as normal and your own version will be used.

| Preinstalled packages | Version |
| :-------------------- | :------ |
| `devtools`            | 2.4.5   |

<details>
<summary>Note for admin</summary>

```r
install.packages("devtools", library = "/opt/R/4.0.3/lib/R/library")
```

</details>

## 4.5. Parallel processing
__Don't use more than 46 cores.__ This leaves 2 cores for other people preparing their stuff. If you want to know how many cores are currently being used (and by whom), open a terminal (`shift + alt + R`) and type `htop`.

Parallel processing clusters can be set up using `cl <- parallel::makeCluster(46)`. Make sure to close your cluster once you're done with it: `parallel::stopCluster(cl)`.

I like using the package [`pbapply`](https://cran.r-project.org/web/packages/pbapply/pbapply.pdf) for parallel simulations, but you can use any method :)

In R, some packages / functions use OpenMP to parallellize underlying C/C++ code. By default, the behaviour of these programs is to use all the cores of a system. An example of such a function is `mgcv::bam()`. If your code uses such functions, you can restrict the number of cores used by putting the following code at the top of your R script:

```r
Sys.setenv("OMP_THREAD_LIMIT" = 46)
```

## 4.6. Additional software

|Software |Location           |
|:--------|:------------------|
| Mplus   | `/opt/mplus/8.11` |
| JAGS    | `/usr/bin/jags`   |
| Matlab  | `/bin/matlab`     |
| Julia   | `/opt/julia`      |
| PyCharm | `/opt/pycharm`    |

### 4.6.1. How to run additional software with a GUI
For Windows:
1. Install [`XMing`](https://sourceforge.net/projects/xming/) and [`Putty`](https://putty.org/). 
2. Run `XMing` on your computer -- this will start an X server to accept incoming display connections
3. Run `putty` with the following configuration:
  - Host name: `mscomputer.fss.uu.nl`
  - Port: `22`
  - under `Connection > SSH > X11`: check `enable X11 forwarding` and set X display location = `localhost:0.0`
4. Click open in putty
5. type in your username and password
6. run the program, for example `pycharm`.

A display should now open. If it does not, contact the administrator.


### 4.6.2. Matlab
To use matlab, you need to activate matlab for your account. For this, you need to create a mathworks account using your uu address on https://nl.mathworks.com/. Then, you can activate matlab for your account together with the administrator. Once activated, you can run matlab `.m` files in the terminal as follows:

```bash
cat path/to/mymatlabfile.m | matlab -nodisplay -nosplash -nodesktop
```

<details>
<summary>Note for admin</summary>
first, connect via ssh with x forwarding, then run

```bash
sudo activate_matlab
```

</details>

If you want to install/use additional software, please send an email to the admin (Erik-Jan; or, better yet, come and find me in C1.22).


## 4.7. GPU
You can use the GPU if you are doing neural network stuff. Please indicate that you are using the GPU in the google sheet as well. 

- Current Nvidia driver installed: `510.39.01`
- Current CUDA version installed: `11.6`