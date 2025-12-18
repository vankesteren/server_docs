This is the documentation for using the Methods & Statistics department compute server.

- [1. Contact information](#1-contact-information)
- [2. Connecting to the server](#2-connecting-to-the-server)
  - [2.1. RStudio server](#21-rstudio-server)
  - [2.2. SSH connection](#22-ssh-connection)
- [3. User account](#3-user-account)
  - [3.1. Updating the password](#31-updating-the-password)
- [4. Using the server](#4-using-the-server)
  - [4.1. Scheduling time](#41-scheduling-time)
  - [4.2. Server specifications](#42-server-specifications)
  - [4.3. Transferring data to the server](#43-transferring-data-to-the-server)
  - [4.4. Storing data](#44-storing-data)
    - [4.4.1. Backing up your data](#441-backing-up-your-data)
  - [4.5. R sessions](#45-r-sessions)
  - [4.6. R packages](#46-r-packages)
  - [4.7. Parallel processing](#47-parallel-processing)
  - [4.8. Additional software](#48-additional-software)
    - [4.8.1. Visual studio code remote programming](#481-visual-studio-code-remote-programming)
    - [4.8.2. How to run additional software with a GUI](#482-how-to-run-additional-software-with-a-gui)


# 1. Contact information
The administrators for the server are:

| name | login | email |
| :--- | :---- | :---- |
| Erik-Jan | erikjan | e.vankesteren1@uu.nl |
| Javier   | javier  | j.garciabernardo@uu.nl |

# 2. Connecting to the server
Connecting to the server is only available in two ways:
- from our department's Utrecht University network
- via Utrecht University [vpn](https://vpn.uu.nl) from anywhere. 

## 2.1. RStudio server
The easiest way to connect is to type in your browser the following URL: [msserver.fss.uu.nl](http://msserver.fss.uu.nl). You wil be greeted with an `RStudio` login window. This works best in google chrome or mozilla firefox.

## 2.2. SSH connection
Alternatively, you can install an ssh client and connect to the server via ssh. In a terminal, enter `ssh username@msserver.fss.uu.nl`, then enter your password, and you will be logged in to the server, at your own home directory. In this way, it is possible to run loads of other software than `R`, for example `python` (preferably through [`uv`](https://docs.astral.sh/uv/)) or `julia` (through [`juliaup`](https://github.com/JuliaLang/juliaup))

# 3. User account
To access the server, you need a login / user account, which is available on request. A user account needs to be manually created for you. Send an email to the admin (Erik-Jan) for this, _with an explanation of why you want to use the computer_. You will receive a default password which you can change when you first log in.

## 3.1. Updating the password
Login to the server, open a terminal within the `rstudio` browser window (`shift + alt + R`), type `passwd <your-user-name>` (for example `passwd erikjan`) and follow the prompts.

# 4. Using the server
To use the server, abide by these rules:

1. Please read the below carefully. 
2. If you aren't sure about something, read again and then _ask_ before doing.
3. If you misuse the server, your account will be suspended.

## 4.1. Scheduling time
Multiple users can connect to the server at the same time. If you are preparing your simulation or just trying out small stuff, you can always login to the server. If you want to run a large simulation, please reserve time for this on the [Google sheet schedule](https://docs.google.com/spreadsheets/d/1YmaAHvosjAvPZCP4mZkHpW-yuWUnar4o5oMbvXBvWIg/edit?usp=sharing).

Please try to keep the maximum reservation time to maximum 1 week in busy periods. The schedule includes a "% use" column, this means either the maximum percentage of cores you will use or the maximum percentage of RAM you will use, whichever is biggest. (Don't worry too much about this, just try to estimate it a little bit by running a test and guesstimating).

You can estimate how much time and memory you will need with the 'Profiling' tool in RStudio on the server.^[For background info on profiling, see e.g. the [Advanced R](https://adv-r.hadley.nz/perf-measure.html#profiling) book. For practical tips on using the Profiling tool, see e.g. the [Posit](https://support.posit.co/hc/en-us/articles/218221837-Profiling-R-code-with-the-RStudio-IDE) support page.] You can also use `htop` in the terminal while running a pilot script, or use the `object.size()` function in the R console to check how big your output will be.

## 4.2. Server specifications

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

## 4.3. Transferring data to the server

RStudio server has a nice "upload" functionality in the files pane, where you can upload individual files or whole folders as a zip file (which is automatically extracted, nice for RStudio projects). 

Another, more robust option for transferring data to the server is through the `scp` (secure copy) command in the terminal of your own computer. For example, to transfer the folder `my_project` in the current working directory to your home folder on the server:

```sh
scp -rC ./my_project username@msserver.fss.uu.nl:~/
```

## 4.4. Storing data
Please exclusively use your home directory (`/data/<your-user-name>/`, or alternatively `~/`), which is on the main drive. No other user has access to your files there (except the admin).

_Example_
```r
my_big_matrix <- matrix(0, 1e4, 1e4)
saveRDS(my_big_matrix, "~/bigfile.rds")
```

> [!CAUTION]
> Some programs or R packages store large amounts of data in the `/tmp` directory on the OS disk. Please make sure that this does not fill up the harddisk or the RStudio server will crash. 
> 
> For example, for `brms` / `cmdstanr`, first create a custom directory in your home directory (e.g., `~/tmp`), and then set `options(cmdstanr_output_dir = "~/tmp")` at the start of your script. Then, make sure to use the `cmdstanr` backend in your call to `brms::brm()`. If you are running `cmdstanr` in parallel, you may need to pass the temporary folder directly during the Markov chain Monte Carlo algorithm (e.g., `cmdstan_model$sample(…, output_dir = "~/tmp")`).

### 4.4.1. Backing up your data

> [!IMPORTANT]
> You are responsible for archiving your data. We provide no guarantees on backing up your data. Consider your home directory as temporary/scratch storage.

After running simulations, it is wise to archive your results somewhere you can access them in case the hard drive of the server somehow breaks or if we need to free up the space. You can do this from the RStudio server by selecting "download" in the files tab. 

It can also be done using the `scp` command from your own computer via the terminal (if you have it installed).

_Example_

To copy the entire folder `simulation_folder` to the local backup folder `local_backup`, the user `testuser` can run the following command:

```bash
scp -rC testuser@msserver.fss.uu.nl:~/simulation_folder local_backup
```

## 4.5. R sessions
When you log in, you start an `R` session. It will remain open until you stop it, even if you close the window. To close the session, click the red 'quit' button in the top right corner. Please always close your `R` session when you are done. The current R version is the following:

```
R version 4.4.2 (2024-10-31) -- "Pile of Leaves"
Copyright (C) 2024 The R Foundation for Statistical Computing
Platform: x86_64-pc-linux-gnu
```

## 4.6. R packages
Everyone gets their own personal `R` package repository automatically in `~/R/packages`. There should not be any interference with package versions.

For robustness, we have installed the [`pak` package manager](https://pak.r-lib.org/). Please use it. It can install packages from CRAN and Bioconductor (`pak::pak("tidyverse")`), GitHub (`pak::pak("vankesteren/pensynth")`) local files (`pak::local_install("~/uploads/my_package")`), and more.

If installation fails for a package, this is likely due to system libraries not being installed. Please try to figure out which system library is needed, you can use the functions: `pak::pkg_sysreqs()` and `pak::sysreqs_check_installed()`. Contact the admin to install the system library for you.

This is what such a warning may look like upon trying to install a package (missing `libmpfr-dev` library):
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
| `pak`                 | 0.8.0.1 |

## 4.7. Parallel processing
Multiple people can be logged into the server at the same time. If you reserve time on the server, you can use it, but please leave at least 4 cores available for other people to prepare their simulations.

If you want to know how many cores are currently being used (and by whom), open a terminal (`shift + alt + R` in RStudio) and type `htop`.

Parallel processing clusters can be set up using various packages, such as `parallel` (built into R), `future`, `parallelly` and more. An example is `cl <- parallel::makeCluster(220)`. 

> [!CAUTION]
> Make sure to close your cluster once you're done with it, for example using `parallel::stopCluster(cl)`. Otherwise, the server will have many processes left open.

I like using the package [`pbapply`](https://cran.r-project.org/web/packages/pbapply/pbapply.pdf) for parallel simulations, but you can use any method :)

In R, some packages / functions use OpenMP to parallellize underlying C/C++ code. By default, the behaviour of these programs is to use all the cores of a system. An example of such a function is `mgcv::bam()`. If your code uses such functions, you can restrict the number of cores used by putting the following code at the top of your R script:

```r
Sys.setenv("OMP_THREAD_LIMIT" = 220)
```

## 4.8. Additional software

|Software |Location           |
|:--------|:------------------|
| Mplus   | `/opt/mplus/8.11` |

- For python, please use [`uv`](https://docs.astral.sh/uv) to manage and run different versions for your own user account. This is portable, fast, and flexible.
- For Julia, use [`juliaup`](https://github.com/JuliaLang/juliaup) for the same reasons.

If you have questions about additional software, send a message to the admin.

### 4.8.1. Visual studio code remote programming
You can also connect to the server via visual studio code. Just install the Remote - SSH extension and use it to connect to the server via SSH.

### 4.8.2. How to run additional software with a GUI
For Windows:
1. Install [`XMing`](https://sourceforge.net/projects/xming/) and [`Putty`](https://putty.org/). 
2. Run `XMing` on your computer -- this will start an X server to accept incoming display connections
3. Run `putty` with the following configuration:
  - Host name: `msserver.fss.uu.nl`
  - Port: `22`
  - under `Connection > SSH > X11`: check `enable X11 forwarding` and set X display location = `localhost:0.0`
4. Click open in putty
5. type in your username and password
6. run the program, for example `pycharm`.

A display should now open. If it does not, contact the administrator.
