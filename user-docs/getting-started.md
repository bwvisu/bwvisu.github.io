# Getting started with bwVisu

**Important: bwVisu does not possess a storage backend. In order to access your data with bwVisu, it must reside on the bwForCluster MLS&WISO or SDS@hd. Data persistence is not guaranteed in your home directory on the bwVisu system itself.**

In order to get started with bwVisu, please proceed as follows:

* Register for the service bwVisu at [http://bwservices.uni-heidelberg.de](http://bwservices.uni-heidelberg.de). It is required to set a service password. Once a service password is set, use this password for the bwvisu login later on.
* Once your registration has been completed, the bwVisu system will set up your user account. This can take up to 10 minutes, so please wait for around ten minutes before proceeding.
* Log in to the bwVisu web frontend at [https://bwvisu-web.urz.uni-heidelberg.de](https://bwvisu-web.urz.uni-heidelberg.de). Your username will be `<site-prefix>_<uni-id>`, e.g. `hd_ab123` for a user from Heidelberg. The password will be your bwVisu service password if set at `bwservices.uni-heidelberg.de`, otherwise it will be your regular uni id password.
* Inside the web frontend, you can start a new job by clicking on an application version in the application list. You will now see the application details. Click the `start new job` button to launch a new job.
* Most bwVisu jobs are Xpra applications. For these, you can connect to the job with your browser by clicking the Xpra link in the job details. Alternatively, you can use the Xpra desktop client with the provided connection data and password.
* Some applications like Vistle require a dedicated desktop client.
* For Xpra applications, you will now see the actual application window as well as an `xterm`-Terminal window inside your browser. Inside the terminal window, a setup program is running that will guide you through the process of connecting to your data on SDS@hd or the bwForCluster MLS&WISO. *Note: If you would like to rerun the setup program at a later time, remove `~/.bwvisu/setup-done`. The program will then start again with your next bwVisu job. Note that connections to SDS@hd or the bwForCluster MLS&WISO persist if you have already configured them previously -- they do not need to be reconfigured.*
* The job will terminate
   * when the job runtime requested in the job submission form has expired
   * when all windows have been closed inside the Xpra popup window (but the job will not terminate if only the popup window itself is closed)
   * When you explicitly cancel the job by clicking the `stop job` button in the job details
