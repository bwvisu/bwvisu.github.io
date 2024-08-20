# RStudio User Guide

## Table of contents
1) [How to add R packages to Rstudio](#how-to-add-r-packages-to-rstudio)


## How to add R packages to RStudio
1) [ssh into Helix](https://wiki.bwhpc.de/e/Helix/Login).
2) Execute `module load math/R/4.2.1`.
3) Load the R environment by executing `R`.
4) Run the following installation command: `install.packages("name-of-the-package")`
   
   If the package will not install because another package could no be installed, then try to run the installation command for that package first. Then retry to install package
   If the installation worked, you'll be able to load the package in bwVisu RStudio.
 
