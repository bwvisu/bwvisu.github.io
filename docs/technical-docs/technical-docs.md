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
   
Jobs are started by substituting variables in batch script templates with values provided by the web frontend and submitting the batch script to Slurm. Also embedded in the batch script is the keyhole client, which will be executed once the batch script runs. It will then request port forwarding from the middleware to its host. The process of launching a job is quite complex; you can find a detailed walkthrough [here](starting-job.md).



## Cluster
The bwVisu backend cluster can in principle be any regular HPC cluster. bwVisu has been designed to be minimally invasive and imposes very few requirements on the cluster. See [here](backend.md) for a detailed description on how to tell the middleware how to talk to an existing cluster, and which prerequisites must be met.

## Applications
