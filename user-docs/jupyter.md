# Jupyter User Guide

## Table of contents
1) [How to add python packages to Jupyter](how-to-add-python-packages-to-jupyter)
2) [How to update the python version](how-to-update-the-python-version)

## How to add python packages to Jupyter

### Create a virtual environment:
There are numerous ways to create virtual environment for python. Here are two ways:
1) Via python's venv module. Open a terminal and type:
   ```
   python3 -m venv <myEnv>
   ```
2) Via miniconda:
  - Load the miniconda module by clicking first on the blue hexagon icon on the left-hand side of Jupyter's start page and then on the "load" Button right of the entry for miniconda in the software module menu: ![Screenshot at 2022-02-10 13-47-37](https://user-images.githubusercontent.com/68850960/153412721-960613c1-ccbd-46a3-922f-bcbf247553a8.png)
    :warning: **Note:** Currently there is a bug preventing the miniconda module to be shown under the "Lodaded Modules" list after loading it. It will be activated anyway, you can check this by opening a terminal and typing `module list`.
  - Open a terminal and type:
    ```
    conda create --name <myenv>
    ```
### Install your packages into your newly created virtual environment:
1) Activate your environment:
    ```
    source <myEnv>/bin/activate
    ```
   or 
    ```
    conda activate <myenv>
    ```
2) Install your packages:
   ```
   pip install -U pip
   pip install <mypackage>
   ```
   or
   ```
   conda install <mypackage>
   ```
### Register your virtual environment for Jupyter:
1) Install the ipykernel package:
   ```
   pip install ipykernel
   ```
   or
   ```
   conda install ipykernel
   ```
2) Register your virtual environment as custom kernel to Jupyter:
   ```
   python3 -m ipykernel install --user --name=<myKernel>
   ```
### Select your newly created kernel `<myKernel>` in Jupyter
  
## How to update the python version
1) Activate the Miniconda module as shown in the [previous section](create-a-virtual-environment).
2) Create a virtual environment with a custom python version via conda:
   ```
   conda create --name <myenv> python=<python version>
   ```
3) [Register](register-your-virtual-environment-for-jupyter) your virtual environment for Jupyter.
4) Select your newly created kernel in Jupyter  
