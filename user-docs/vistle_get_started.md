# Vistle User Guide #

Using Vistle requires a local installation of the Vistle client. If you have it installed already, you can skip part A and continue directly with part B.

## Part A - Obtaining the Vistle client
There are three options for setting up the Vistle client:

**a)** Using Singularity
If you have Singularity installed on your machine, download the repository containing Vistle definition files and follow the instruction as described there 
https://github.com/vistle/singularity

**b)** Clone from GitHub
https://github.com/vistle/vistle
Make sure you have installed all dependencies listed on this site. Then follow the  instructions there

**c)** MacOS only: Install using Homebrew
> brew install hlrs-vis/tap/vistle

## Part B: Connecting to the server
Start a new Vistle job on bwVisu.
Once the job is running, you can find the servers address and port listed in the job information section:  tcp://\<ip\>:\<port\>

Follow the instruction in either a) or b), depending on if you are using Singularity.

**a)** Starting Vistle from the Singularity image
1.	Go to the singularity directory and enter the Vistle client image:
      > singularity shell --nv vistle-client.sif
2.	Inside: Set the Vistle key variable
      > Singularity> export VISTLE_KEY=0000
3.	Connect to the server by inserting the IP and Port from the bwVisu job information
      > Singularity> /usr/bin/vistle -c \<ip\>:\<port\>
      
**b)** Starting Vistle if installed via GitHub or Homebrew
1.  On your client machine, go to the Vistle directory and set the key variable:
      > export VISTLE_KEY=0000
2.	Connect to the server by inserting the IP and Port from the bwVisu job information
      > vistle -c \<ip\>:\<port\>

The Vistle GUI should now show up and you are all set to use Vistle. To verify the connection to the server, go to the module browser on the right hand side of the GUI. You should see two drop down lists: one for your client machine and one for the remote server, depending on where you want to start a module. You can now create your module map!
