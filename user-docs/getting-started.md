# Getting started with bwVisu

In order to get started with bwVisu, please proceed as follows:

* In order to use bwVisu, it is required to be a user of the [bwForCluster MLS&WISO](https://wiki.bwhpc.de/e/BwForCluster_User_Access), or a user of [SDS@hd](https://wiki.bwhpc.de/e/Sds-hd_user_access).
* Register for the service bwVisu at [http://bwservices.uni-heidelberg.de](http://bwservices.uni-heidelberg.de). It is required to set a service password. Once a service password is set, use this password for the bwvisu login later on. It is also mandatory to register a token for two factor authentication at the bwservices site. More information about registering a token can be found [here](https://wiki.bwhpc.de/e/BwForCluster_MLS%26WISO_Production_2FA_tokens).
* Once your registration has been completed, the bwVisu system will set up your user account. This can take up to 10 minutes, so please wait for around ten minutes before proceeding.
* Log in to the bwVisu web frontend at [https://bwvisu-web.urz.uni-heidelberg.de](https://bwvisu-web.urz.uni-heidelberg.de). Your username will be `<site-prefix>_<uni-id>`, e.g. `hd_ab123` for a user from Heidelberg. The password will be your bwVisu service password set at `bwservices.uni-heidelberg.de`, and your registered device will be used as second factor.
* Inside the web frontend, you can start a new job by clicking on an application version in the application list. You will now see the application details. Click the `start new job` button to launch a new job.
* Most bwVisu jobs are Xpra applications. For these, you can connect to the job with your browser by clicking the Xpra link in the job details.
* Some applications like Vistle require a dedicated desktop client.
* The job will terminate
   * when the job runtime requested in the job submission form has expired
   * When you explicitly cancel the job by clicking the `stop job` button in the job details
   * Xpra jobs: when all windows have been closed inside the Xpra window (but the job will not terminate if only the Xpra window itself is closed)
