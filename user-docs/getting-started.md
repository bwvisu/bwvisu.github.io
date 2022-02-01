# Getting started with bwVisu

**Important: bwVisu does not possess a storage backend. In order to access your data with bwVisu, it must reside on the bwForCluster MLS&WISO or SDS@hd. Data persistence is not guaranteed in your home directory on the bwVisu system itself.**

In order to get started with bwVisu, please proceed as follows:

* In order to use bwVisu, it is required to be part of a [MLS&WISO Rechenvorhaben](https://wiki.bwhpc.de/e/BwForCluster_User_Access), or to be a member or a [SDS@hd Storageproject](https://wiki.bwhpc.de/e/Sds-hd_user_access).
* Register for the service bwVisu at [http://bwservices.uni-heidelberg.de](http://bwservices.uni-heidelberg.de). It is required to set a service password. Once a service password is set, use this password for the bwvisu login later on. It is also mandatory to register a token for two factor authentication at the bwservices site. More information about registering a token can be found [here](https://wiki.bwhpc.de/e/BwForCluster_MLS%26WISO_Production_2FA_tokens).
* Once your registration has been completed, the bwVisu system will set up your user account. This can take up to 10 minutes, so please wait for around ten minutes before proceeding.
* Log in to the bwVisu web frontend at [https://bwvisu-web.urz.uni-heidelberg.de](https://bwvisu-web.urz.uni-heidelberg.de). Your username will be `<site-prefix>_<uni-id>`, e.g. `hd_ab123` for a user from Heidelberg. The password will be your bwVisu service password set at `bwservices.uni-heidelberg.de`, and your registered device will be used as second factor.
* Inside the web frontend, you can start a new job by clicking on an application version in the application list. You will now see the application details. Click the `start new job` button to launch a new job.
* Most bwVisu jobs are Xpra applications. For these, you can connect to the job with your browser by clicking the Xpra link in the job details. Alternatively, you can use the Xpra desktop client with the provided connection data and password.
* Some applications like Vistle require a dedicated desktop client.
* The job will terminate
   * when the job runtime requested in the job submission form has expired
   * when all windows have been closed inside the Xpra popup window (but the job will not terminate if only the popup window itself is closed)
   * When you explicitly cancel the job by clicking the `stop job` button in the job details
