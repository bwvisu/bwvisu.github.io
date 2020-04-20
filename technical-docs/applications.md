# bwVisu applications

At the core of bwVisu are the provided visualization applications. These are containerized using the container solution Singularity.

## Restrictions on images
Applications in bwVisu have only a few restrictions. These being namely:

1. The RUN command needs to run the graphical application directly as we want to execute `singularity run IMAGE_NAME` with the appropriate executable set as run command. This should be the case for many applications already available on public registries.
2. Images do not need to contain the Nvidia driver as it will be provided by the HPC system. However MPI needs to match the version of the system the application is running and needs to be inside of the image.
3. Generally speaking, the functionality of the image should be checked. If it did not run on you local machine do not expect it to work with bwVsu.

## Modified Xorg
bwVisu provides every job with an accelerated Xorg as this is necessary to have hardware acceleration for your application. Xorg itself will be provided by an image (probably named `bwvisu_xorg.sif`). This Xorg is modified to patch out the need for virtual consoles and can be started by non-root users. This is necessary for providing an X server for every job so that users do not need to share one. For more information see [this article](https://devblogs.nvidia.com/hpc-visualization-nvidia-tesla-gpus/) and [this issue](https://gitlab.freedesktop.org/xorg/xserver/issues/635).


## How does bwVisu run applications?
There are a number of steps involved before a user can actually interact with a visualization application. Here is a rough breakdown of the steps.

1. The user configures his job in the webfrontend by choosing an application and the job configuration. This sets environment variables like the name of the application. Information about the available applications is stored in `app bundles`. See [here](webfrontend.md) for more information on how to provide these app bundles.
2. The job information is sent to the bwVisu runner. The runner will execute a batch script in the name of the user. The important steps here are the execution of the Xorg image after which the the process sleeps until the X server generates a unix domain socket for a random display number inside of a job specific temporary directory.
3. After the unix domain socket is detected Xpra will use that display and start the actual application as child process. Xpra will use the `singularity run` command with a couple of other options.
4. After some time the connection information will be displayed in the webfrontend.

