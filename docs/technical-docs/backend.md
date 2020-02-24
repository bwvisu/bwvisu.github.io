# bwVisu backend configuration

bwVisu interacts with an HPC cluster that serves as backend. This is done via a special middleware web application, that provides a REST API for the web frontend, and manages the HPC cluster via ssh.
In order to use a given HPC cluster for bwVisu, it must therefore be configured for the middleware and the bwVisu application images must be available on the cluster.

## bwVisu middleware backend requirements

The bwVisu middleware distinguished between 3 different types of nodes:

* Compute nodes - Nodes where the jobs will be running
* Command nodes - Nodes from which jobs are managed
* Gateway nodes - Nodes that are used as 'data gateways' which the user can use connect to the jobs.

These node types refer to feature sets, not to physically distinct machines - for example, a machine could be compute, command and gateway node at the same time.

The machine on which the bwVisu middleware is running must be reachable from the bwVisu webfrontend and the compute nodes. It does not necessarily have to run on compute, command or gateway nodes.

### Command nodes
These nodes are used to manage jobs. They need to satisfy the following requirements:
* Slurm commands `sbatch`, `squeue`, `scancel` are available
* There exists a user that the middleware can use to login via ssh
* The middleware ssh key must be installed for this user, such that the middleware can login without password.
* The middleware user has passwordless sudo permissions for sbatch, squeue, scancel

### Compute nodes
These are nodes on which the Slurm from the command nodes executes jobs.
* python3 must be installed, i.e. `python3` command must work
* Middleware must be reachable from compute nodes
* For the current bwVisu images: singularity, sshfs must be installed
* For some `$PREFIX` there exist private directories `$PREFIX/$USER` per user to store the logfiles

### Gateway nodes
These are nodes through which the user connections will be forwarded to the jobs.
* Gateway nodes must be publicly reachable
* Compute nodes must be reachable
* `iptables` must be installed
* netcat must be available, in particular `nc -z` command in order to find unused ports and for health-checking of jobs.
* A middleware user must exist that the middleware can use to login via ssh (may or may not be different from the user for the command nodes)
* The middleware ssh key must be installed for this user such that the middleware can login without password.
* Middleware user must have passwordless sudo permissions for `iptables`.
* A large contiguous block of free ports that can be allocated to bwVisu is required (by default, 50000-60000). These ports will be used to forward connections from their users to their jobs.


Gateway nodes will typically be login nodes or VMs running on login nodes.

### Example Middleware configuration for typical HPC cluster

```
{
    "bwvisu-home-prefix" : "/shared/home",
    "gateway-nodes" : [
        {
            "internal-ip" : "10.100.0.101",
            "external-ip" : "129.206.9.248",
            "user" : "bwvisurunner",
            "port" : 22
        },
        {
            "internal-ip" : "10.100.0.102",
            "external-ip" : "129.206.9.253",
            "user" : "bwvisurunner",
            "port" : 22
        }

    ],
    "command-nodes" : [
        {
            "ip" : "129.206.9.248",
            "user" : "bwvisurunner",
            "port" : 22                                                                                                                                        
        },
        {
            "ip" : "129.206.9.253",
            "user" : "bwvisurunner",
            "port" : 22
        }
    ],
    "keyholed-servers" : [ { "ip": "129.206.7.86", "port" : 80} ],
    "keyholed-port-range" : { "min" : 50000, "max" : 60000 },
    "keyholed-max-forwarders-per-job" : 100,
    "etcd-servers" : [ { "ip": "127.0.0.1", "port" : 2379} ],
    "debug" : {
        "keyholed-static-forwarding" : "False"
    },
    "forwarding-hostname-suffix" : "-ib0"
}

```

### bwVisu applications on the backend

In order to start the bwVisu application images on the backend, you need to have installed
* `singularity`
* `Xpra`
* the bwVisu Xorg singularity image

