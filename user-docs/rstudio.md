# RStudio User Guide

## Table of contents
1) [How to change the R version in RStudio](how-to-change-the-r-versin-in-rstudio)
   1) [Change the R version to one of the available versions on Helix](change-the-r-version-to-one-of-the-available-versions-on-helix)
   2) [Change the R version to a version unavailable on Helix](change-the-r-version-to-a-version-unavailable-on-helix)      
3) [How to add R packages to Rstudio](#how-to-add-r-packages-to-rstudio)

## How to change the R version in RStudio
By default, the default R version on Helix is used. You can view the current default version by executing `module -t -d avail 2>&1 | grep math/R` on Helix.
### Change the R version to one of the available versions on Helix
1) Open or create the file `.bashrc` in your home directory, either by [opening a shell on Helix](https://wiki.bwhpc.de/e/Helix/Login) or by starting the RStudio app on bwVisu and  opening the file there.
2) Add the following line to your .bashrc file:
   ```bash
   export R_MODULE_VERSION=<available R version on Helix>
   ```
   for example `export R_MODULE_VERSION=4.2.1`.
3) (Re)start RStudio 
### Change the R version to a version unavailable on Helix
1) Open a shell on Helix by [sshing into Helix](https://wiki.bwhpc.de/e/Helix/Login) or by starting the RStudio app on bwVisu and switching to the Terminal tab:
2) [Download](https://cran.r-project.org/doc/FAQ/R-FAQ.html#How-can-R-be-obtained_003f-1) your desired R version, [unpack it](https://cran.r-project.org/doc/manuals/r-release/R-admin.html#Getting-and-unpacking-the-sources-1) and change into the unpacked directory:
   ```bash
   tar -xf R-x.y.z.tar.gz
   cd R-x.y.z.tar.gz
   ```
   where `x.y.z` denotes the version of your download R installation files.
3) Patch the configuration file to make it compatible with your shell environment on Helix:
   ```bash
   sed '/date-time conversions do not work in 2020/s/^/: #/' -i configure
   ```
4) Install R into a chosen location in your home directory:
   ```bash
   ./configure prefix=$HOME/path/in/you/home/directory
   make
   make install
   ```
5) Make your installed R version the default by exporting the location of its binary to your `PATH`: Open or create the file `.bashrc` in your home directory and add the following line: 
   ```bash
   export "$PATH=/path/to/your/R/installation/directory/bin:$PATH"
   ```
   Please make sure that the suffix `/bin` is added to the path to the installation location of your custom R version. This is the default location where the binaries of R will be usually installed.
6) Test the installation: Reload your bash environment and obtain the version of the R binary in your path: 
   ```bash
   source $HOME/.bashrc && R --version
   ```
   If the displayed version number is equal to the one of your custom R installation, then the installation was successfull and your custom R version will be automatically loaded in RStudio.
7) (Re)start RStudio
## How to add R packages to RStudio
1) Open a shell on Helix by [sshing into Helix](https://wiki.bwhpc.de/e/Helix/Login) or by starting the RStudio app on bwVisu and switching to the Terminal tab.
2) Execute 
   ```bash
   module load math/R/4.2.1
   ```
3) Load the R environment by executing
   ```bash
   R
   ```
5) Run the following installation command:
   ```bash
   install.packages("name-of-the-package")
   ```
   
   If the package will not install because another package could no be installed, try to run the installation command for that package first. After that, retry to install package.
   If the installation worked, you'll be able to load the package in RStudio.
 
