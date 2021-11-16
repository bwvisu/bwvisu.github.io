# Jupyter User Guide #

## Create new kernel for Jupyter

Open a terminal and an create a virtual environment:
```
{
python3 -m venv <myEnv>
source <myEnv>/bin/activate
}

```

Install packages in virtual environment:
```
{
pip install --upgrade pip
pip install ipykernel
pip install <myPackage> 
}

```

Create new kernel:
```
{
python3 -m ipykernel install --user --name=<myKernel>
}

```

Start <myKernel> via Jupyter launcher
