# Jupyter User Guide #

## Create customized kernel for Jupyter

Open a terminal and create virtual environment `<myEnv>`:
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
