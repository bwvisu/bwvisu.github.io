# Jupyter User Guide

## Table of contents
1) [How to add python packages to Jupyter](#how-to-add-python-packages-to-jupyter)
   1) [Via virtual environments](#via-virtual-environments)
   2) [Via miniconda](#via-miniconda)
2) [How to update the python version](#how-to-update-the-python-version)
3) [How to use R in Jupyter](#how-to-use-r-in-jupyter)
4) [Frequently Asked Questions](#faq)

## How to add python packages to Jupyter

### Via virtual environments
1) Open a terminal and type:
   ```
   python3 -m venv <myEnv>
   ```
1) Activate your environment:
   ```
   source <myEnv>/bin/activate
   ```
2) Install your packages:
   ```
   pip install -U pip
   pip install <mypackage>
   ```
3) Install the ipykernel package:
   ```
   pip install ipykernel
   ```
2) Register your virtual environment as custom kernel to Jupyter:
   ```
   python3 -m ipykernel install --user --name=<myKernel>
   ```
### Via miniconda
1) Load the miniconda module by clicking first on the blue hexagon icon on the left-hand side of Jupyter's start page and then on the "load" Button right of the entry for miniconda in the software module menu: ![Screenshot at 2022-02-10 13-47-37](https://user-images.githubusercontent.com/68850960/153412721-960613c1-ccbd-46a3-922f-bcbf247553a8.png)    

2) Open a terminal and complete the miniconda setup with:
    ```
    source $MINICONDA_HOME/etc/profile.d/conda.sh 
    ```
3) Ceate a conda environment:
    ```
    conda create --name <myenv>
    ```
4) Activate your environment:
    ```
    conda activate <myenv>
    ```
5) Install your packages:
   ```
   conda install <mypackage>
   ```
6) Install the ipykernel package:
   ```
   conda install ipykernel
   ```
7) Register your virtual environment as custom kernel to Jupyter:
   ```
   python3 -m ipykernel install --user --name=<myKernel>
   ```
  
## How to update the python version
1) Activate the Miniconda module as shown in the [previous section](#via-miniconda).
2) Create a virtual environment with a custom python version via conda:
   ```
   conda create --name <myenv> python=<python version>
   ```
3) Install the ipykernel package:
   ```
   conda install ipykernel
   ```
4) Register your virtual environment as custom kernel to Jupyter:
   ```
   python3 -m ipykernel install --user --name=<myKernel>
   ```
5) Select your newly created kernel in Jupyter

## How to use R in Jupyter

1) On the cluster:
   ```
   $ module load math/R
   $ R
   > install.packages('IRkernel')
   ```
2) On bwVisu:
   1) Start Jupyter App
   2) In left menu: load math/R
   3) Open Console:
    ```
    $ R
    > IRkernel::installspec(displayname = 'R 4.2')
    ```
   4) Start kernel 'R 4.2' as console or notebook

## FAQ

1) Question: My conda commands are interrupted with message 'Killed'. What can I do?

   Answer: Request more memory when starting Jupyter. In the job settings open 'Cluster Settings' and look for the option 'Memory / node'.

2) Question: How can I navigate to my SDS@hd folder in the file browser?

   Answer: Open a terminal and set a sympolic link to your SDS@hd folder in your home directory. For example:
   ```
   $ cd $HOME
   $ mkdir sds-hd
   $ cd sds-hd
   $ ln -s /mnt/sds-hd/sd16a001 sd16a001
   ```
