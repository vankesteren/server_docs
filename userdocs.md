This is the documentation for using the Methods & Statistics department compute server.

- [1. Connecting to the server](#1-connecting-to-the-server)
  - [1.1. RStudio server](#11-rstudio-server)
  - [1.2. SSH connection](#12-ssh-connection)
- [2. User account](#2-user-account)
  - [2.1. Updating the password](#21-updating-the-password)
- [3. Using the server](#3-using-the-server)
  - [3.1. Server specifications](#31-server-specifications)
  - [Transferring data to the server](#transferring-data-to-the-server)
  - [3.2. Storing data](#32-storing-data)
    - [3.2.1. Backing up your data](#321-backing-up-your-data)
  - [3.3. R sessions](#33-r-sessions)
  - [3.4. R packages](#34-r-packages)
  - [3.5. Parallel processing](#35-parallel-processing)
  - [3.6. Additional software](#36-additional-software)
    - [3.6.1. How to run additional software with a GUI](#361-how-to-run-additional-software-with-a-gui)
    - [3.6.2. Matlab](#362-matlab)
  - [3.7. GPU](#37-gpu)


# 1. Connecting to the server
Connecting to the server is only available in two ways:
- from our department's Utrecht University network
- via Utrecht University [vpn](https://vpn.uu.nl) from anywhere. 

## 1.1. RStudio server
The easiest way to connect is to type in your browser the following URL: [msserver.fss.uu.nl](http://msserver.fss.uu.nl). You wil be greeted with an `RStudio` login window. This works best in google chrome or mozilla firefox.

## 1.2. SSH connection
Alternatively, you can install an ssh client and connect to the server via ssh. In a terminal, enter `ssh username@msserver.fss.uu.nl`, then enter your password, and you will be logged in to the server, at your own home directory. In this way, it is possible to run loads of other software than `R`, for example `python` (preferably through [`uv`](https://docs.astral.sh/uv/)) or `julia` (through [`juliaup`](https://github.com/JuliaLang/juliaup)).

# 2. User account
To access the server, you need a login / user account, which is available on request. A user account needs to be manually created for you. Send an email to the admin (Erik-Jan) for this, _with an explanation of why you want to use the computer_. You will receive a default password which you can change when you first log in.

## 2.1. Updating the password
Login to the server, open a terminal within the `rstudio` browser window (`shift + alt + R`), type `passwd <your-user-name>` (for example `passwd erikjan`) and follow the prompts.

# 3. Using the server
To use the server, abide by these rules:

1. Please read the below carefully. 
2. If you aren't sure about something, read again and then _ask_ before doing.
3. If you misuse the server, your account will be suspended.

Multiple users can connect to the server at the same time. If you are preparing your simulation or just trying out small stuff, you can always login to the server. If you want to run a large simulation, please reserve time for this on the [Google sheet schedule](https://docs.google.com/spreadsheets/d/1YmaAHvosjAvPZCP4mZkHpW-yuWUnar4o5oMbvXBvWIg/edit?usp=sharing).

## 3.1. Server specifications

The server is a virtual server running on quite serious hardware. The server can be scaled up to the following:
```
CPU     :  2x INTEL(R) XEON(R) PLATINUM 8580
Threads :  240
Memory  :  1TB
GPU     :  None
Storage :  Main disk : 2.0T /data
           OS disk   : 58G  /
```

However, by default the server has fewer threads (usually around 224). To check from R how many threads are available, run `parallel::detectCores()`.

## Transferring data to the server

RStudio server has a nice "upload" functionality in the files pane, where you can upload individual files or whole folders as a zip file (which is automatically extracted, nice for RStudio projects).

## 3.2. Storing data
Please exclusively use your home directory (`/data/<your-user-name>/`, or alternatively `~/`), which is on the main drive. No other user has access to your files there (except the admin).

_Example_
```r
my_big_matrix <- matrix(0, 1e4, 1e4)
saveRDS(my_big_matrix, "~/bigfile.rds")
```

> [!NOTE]
> Some programs or R packages store large amounts of data in the `/tmp` directory on the OS disk. Please make sure that this does not fill up the harddisk or the RStudio server will crash. 
> 
> For example, for `brms` / `cmdstanr`, first create a custom directory in your home directory (e.g., `~/tmp`), and then set `options(cmdstanr_output_dir = "~/tmp")` at the start of your script. Then, make sure to use the `cmdstanr` backend in your call to `brms::brm()`.

### 3.2.1. Backing up your data

> You are responsible for archiving your data. We provide no guarantees on backing up your data. Consider your home directory as temporary/scratch storage.

After running simulations, it is wise to archive your results somewhere you can access them in case the hard drive of the server breaks. You can do this from the RStudio server by selecting "download" in the files tab. 

It can also be done using the `scp` command from your own computer via the terminal (if you have it installed).

_Example_

To copy the entire folder `simulation_folder` to the local backup folder `local_backup`, the user `testuser` can run the following command:

```bash
scp -rC testuser@msserver.fss.uu.nl:~/simulation_folder local_backup
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

## 3.3. R sessions
When you log in, you start an `R` session. It will remain open until you stop it (red button in the top right corner). Please close your `R` session when you are done. The R version is the following:

```
R version 4.2.3 (2023-03-15) -- "Shortstop Beagle"
Copyright (C) 2023 The R Foundation for Statistical Computing
Platform: x86_64-pc-linux-gnu (64-bit)
```

## 3.4. R packages
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

## 3.5. Parallel processing
__Don't use more than 46 cores.__ This leaves 2 cores for other people preparing their stuff. If you want to know how many cores are currently being used (and by whom), open a terminal (`shift + alt + R`) and type `htop`.

Parallel processing clusters can be set up using `cl <- parallel::makeCluster(46)`. Make sure to close your cluster once you're done with it: `parallel::stopCluster(cl)`.

I like using the package [`pbapply`](https://cran.r-project.org/web/packages/pbapply/pbapply.pdf) for parallel simulations, but you can use any method :)

In R, some packages / functions use OpenMP to parallellize underlying C/C++ code. By default, the behaviour of these programs is to use all the cores of a system. An example of such a function is `mgcv::bam()`. If your code uses such functions, you can restrict the number of cores used by putting the following code at the top of your R script:

```r
Sys.setenv("OMP_THREAD_LIMIT" = 46)
```

## 3.6. Additional software

|Software |Location           |
|:--------|:------------------|
| Mplus   | `/opt/mplus/8.11` |
| JAGS    | `/usr/bin/jags`   |
| Matlab  | `/bin/matlab`     |
| Julia   | `/opt/julia`      |
| PyCharm | `/opt/pycharm`    |

### 3.6.1. How to run additional software with a GUI
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


### 3.6.2. Matlab
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


## 3.7. GPU
You can use the GPU if you are doing neural network stuff. Please indicate that you are using the GPU in the google sheet as well. 

- Current Nvidia driver installed: `510.39.01`
- Current CUDA version installed: `11.6`