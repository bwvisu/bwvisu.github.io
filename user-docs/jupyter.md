# Jupyter User Guide #

## Create customized kernel for Jupyter


***Note:***
Follow the steps below only in the Jupyter-Shell. Virtual environments created in an ssh-shell on the BwForCluster will not be detected by Jupyter.

In Jupyter, open a new terminal via the menu (File -> New -> Terminal) or via the launcher (File -> New Launcher, or press Ctrl-Shift-L) and create a virtual environment `<myEnv>`:
```
python3 -m venv <myEnv>
source <myEnv>/bin/activate
```

Install packages in virtual environment:
```
pip install --upgrade pip
pip install ipykernel
pip install <myPackages> 
```

Create new kernel:
```
python3 -m ipykernel install --user --name=<myKernel>
```

Start new kernel `<myKernel>` via Jupyter launcher.
