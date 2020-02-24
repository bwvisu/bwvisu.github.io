# bwVisu technical documentation

## Architecture

The bwVisu stack consists of multiple parts:
* A webfrontend provides a user interface for users to interact with bwVisu. The webfrontend in turn interacts with the bwVisu middleware via a [REST API](middleware-api.md);
* A middleware that reacts to requests from the web frontend and manages bwVisu jobs on a cluster;
* An HPC-like cluster running Slurm where the bwVisu application images are available and can be executed using singularity. bwVisu integrates very well with existing HPC cluster deployments and imposes very few requirements on the cluster.
* The bwVisu application images

The rough architecture therefore looks something like this:
```

User <-------> Webfrontend <------> Middleware <-----> HPC Cluster
       HTTPS                HTTPS                SSH
                            (REST API)
```


bwVisu distinguishes three types of nodes in the cluster (more details [here](backend.md)):
* Compute nodes where jobs will run
* Command nodes that the bwVisu middleware will use to manage jobs (typically, these are login nodes)
* Gateway nodes that are used to channel the visualization stream to the user; a port forwarding mechanism allows the user to connect to a port on these nodes which will then be forwarded to the compute node of the MPI master rank.

Note that in this setup, the actual connection between user and application for the visualization data does *not* go through the web frontend. Instead, the web frontend only provides the user with a link which points to the properly forwarded port on one of the gateway nodes. The actual remote visualization software (for most applications we use Xpra) runs as part of the job.
This means that **the load on the web frontend can be expected to be very light, as the web frontend and middleware merely manage jobs**.
The following image illustrates the bwVisu architecture in more detail:

![bwVisu architecture](../img/architecture.png)


## Webfrontend

## Middleware
A custom middleware provides an interface for the web frontend to manage jobs. The middleware, called bwVisu *runner* is written in python and exposes its features via a flask-based HTTP REST API to the web frontend. 

The middleware is designed to be minimally invasive and can be deployed either on nodes of the backend cluster or on an external system. For job management, the middleware communicates via `ssh` with a set of command nodes, where it starts, stops and monitors jobs with the Slurm commands `sbatch, squeue, scancel`. If a command node cannot be reached (e.g. because the host is down), the middleware will attempt to use another command node.

Additionally, the middleware uses `ssh` to access a set of `gateway nodes` where it establishes iptables forwarding rules of network traffic from ports of the gateway node to the individual compute nodes where the user application runs. This allows the user to connect to his job simply by connecting to a specific network port on one of the gateway nodes. This is necessary because the compute nodes of an HPC cluster are typically not directly available from the internet.
Authentication of users connecting to a port is handled by the user application (e.g. Xpra). 
In order to scale to many concurrent jobs, the middleware automatically distributes the port forwarding assignments (and hence, network load) of the jobs across the available gateway nodes.

The state of jobs is continuously monitored from the gateway nodes. This allows the middleware to provide information to the web frontend whether the job is ready to interact with the user, or whether the user still has to wait before the job can be accessed (e.g., because it is queued and not yet running, or because it is running but still in an initial start-up phase where network connections are not yet accepted).

All state of the middleware is stored in an *etcd* database, which is a distributed key-value store. Because of the distributed design of `etcd`, it is easy to operate in a redundant manner and hence allows for a resilient, highly-available deployment of the middleware.

The requirements that the bwVisu middleware imposes on the cluster are designed to be as low as possible: The middleware must be granted ssh access to a dedicated user account, and this account must be able to run Slurm commands on the command nodes and iptables on the gateway nodes ([details here](backend.md)).
Since the installation of the bwVisu middleware is the only requirement to use a given HPC cluster for bwVisu, extending an existing HPC cluster with bwVisu's remote visualization capabilities is easy and does not necessitate any changes to the existing cluster configuration.

The middleware consists of four parts:
* the actual *runner* consists of two parts:
   * Flask-based REST API for the communication with the Web frontend
   * Background *watcher* process that periodically queries the HPC scheduler on the cluster for information on the running jobs. This is used to update the list of running jobs and the state of the jobs.
* the *keyhole* service that manages port forwarding rules, which also consists of two halves:
   * Flask-based REST API which running jobs can use to request a port forwarding rule
   * Background *watcher* process that checks if a service has actually started listening on assigned ports. This is done using `nc -z` from the gateway nodes, and is necessary because after the job has started, it may still take some time for e.g. an Xpra job to be ready to accept user connections. During this time, the job should not yet be advertised as `available` to the user because the user will not be able to successfully connect. Therefore it is necessary to actually test if the ports accept connections. This mechanism can also be interpreted as a *health check* for running jobs. The keyhole watcher also makes sure that iptables forwarding rules that belong to expired jobs are removed again.
   
### Starting a job

While e.g. cancelling a job is a relatively straight-forward operation of simply invoking `scancel <jobid>` (after which the *watcher* will detect that the job is no longer running and remove it from the job list in the database), starting a job is a highly non-trivial operation. It is therefore worthy for the understanding of the middleware to discuss in detail how this works. Starting a job involves the following steps:

1. The web frontend submits a request to start a job via the middleware REST API. This request includes ([detailed information](middleware-api.md) on the API):
   * The name of the user that owns the job
   * The name of the *batch script template*. The middleware will attempt to load this file from the `$INSTALL_DIR/templates` directory. A batch script template is a template for generating a submission script for the HPC scheduler. Templates are in the mustache templating format and hence can contain variables (for example, the name of the launched singularity image) that will be substituted by the middleware in the next step.
   * A dictionary of variables and their values that the middleware should replace in the batch script template. Notable variables include the name of the singularity image, the number of nodes or expected runtime of the job.
2. An authentication token is generated for the job and deposited in the middleware's database. This authentication token is used later on to ensure that only jobs that have been submitted through the bwVisu web frontend are allowed to request port forwarding rules.
3. The middleware turns the batch script template into an actual batch script by
   * Substituting all variables in the template with the values provided by the web frontend. Because these values enter a shell script, they are sanitized for any characters that could be used to inject additional commands ([more information](middleware-api.md))
   * Expanding lines of the form `#KEYHOLE-REQUEST-PORTS N` into code that will request iptables forwarding of `N` ports from the gateway to the compute node. This line is allowed to occur at most once in a batch script template. In particular, the middleware will embed at the location of this line the code of the python keyhole client script using the bash heredoc format. This script will then be extracted and invoked when the bash script is executed, and will then communicate with the keyhole service to negotiate the forwarding of the ports. Requesting port forwarding can only be done once the job is running because before that, it is not clear which nodes the HPC scheduler will assign for this job. After this line, the environment variables `KEYHOLE_PORT<I>`, `KEYHOLE_GATEWAY<I>`, `KEYHOLE_EXTERNAL_PORT<I>` will be defined in the script. `<I>` corresponds to the index of the port.
   * The environment in which the keyhole client is executed must contain the previously mentioned authentication token. For this purpose, it is also embedded in the bash script.
4. The job is submitted to the HPC scheduler. This is done by piping the generated bash script directly through ssh into `sbatch`, to avoid batch scripts containing authentication tokens being stored persistently on the filesystem. The middleware parses the `sbatch` output to extract the job id and stores it in the database for internal use.
5. The request has finished at this point, and the job id is returned to the web frontend. The returned job id is however not the Slurm job id, but a bwVisu job id (which the middleware can translate back into the Slurm job id if necessary using the stored id in the database). Distinguishing these ids is necessary because in general, not all jobs that Slurm manages are bwVisu jobs if the cluster is not dedicated exclusively to bwVisu. The middleware therefore must enumerate its jobs independently of Slurm.
6. Although the request has finished and the web frontend can present to the user that the job was submitted, the job is not yet running. Assume that at some point the HPC scheduler decides to execute the job and executes the batch script. At this point, the keyhole client will be executed and contact the middleware to request forwarding of ports. It will submit the hostnames of the nodes in this request, which requires parsing the `SLURM_NODELIST` environment variable.
7. The keyhole service performs an HMAC authentication of the request and compares the supplied authentication token from the request with the stored token for this job in the database. Jobs are only allowed to request port forwarding once to prevent DOS attacks, and the maximum number of forwarded ports per job is also limited for the same reason.
8. The middleware now tries to find suitable ports that are available for forwarding. For this purpose, it checks the database for entries of already assigned ports. If it has found a port that appears unused, it makes sure by invoking `nc -z $HOST $PORT` via ssh from a gateway node that indeed no service is listening on the port. The middleware will attempt to find ports that are unused on all involved hosts, such that the application can rely on being able to use this port e.g. for all MPI processes on multiple nodes. 
9. `iptables` is invoked on a gateway node to establish the port forwarding. The database is updated to mark the port as used and the forwarding rule as active.
10. The port and gateway ip is then returned to the keyhole client executed by the batch script, and the keyhole client will make sure that this information is available as environment variable for the remainder of the batch script.
11. The batch script continues executing and the actual application starts.
12. At some point later, the keyhole watcher will detect, again using `nc -z`, that an assigned port of the job is now open and the application is listening for connections. The keyhole watcher marks the job as `'available'`, and the middleware will inform the web frontend upon the next query that the user is now able to connect to the application using the assigned port and gateway ip.
13. The web frontend shows the user that the job is available and displays information on how to connect.



## Cluster
The bwVisu backend cluster can in principle be any regular HPC cluster. bwVisu has been designed to be minimally invasive and imposes very few requirements on the cluster. See [here](backend.md) for a detailed description on how to tell the middleware how to talk to an existing cluster, and which prerequisites must be met.

## Applications
