# bwVisu backend configuration

bwVisu interacts with an HPC cluster that serves as backend. This is done via a special middleware web application, that provides a REST API for the web frontend, and manages the HPC cluster via ssh.
In order to use a given HPC cluster for bwVisu, it must therefore be configured for the middleware and the bwVisu application images must be available on the cluster.

## bwVisu middleware backend requirements

The bwVisu middleware distinguished between 3 different types of nodes:

* Compute nodes - Nodes where the jobs will be running
* Command nodes - Nodes from which jobs are managed
* Gateway nodes - Nodes that are used as 'data gateways' which the user can use to connect to the jobs.

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

### Configuring the middleware for an HPC cluster

The middleware is configured using a config file that is searched in the following locations (sorted by decreasing priority):

1. `$INSTALL_DIR/config.json`
2. `$INSTALL_DIR/../etc/config.json`
3. `/etc/runner/config.json"`
4. `/opt/runner/etc/config.json`

Example configuration file content, assuming 
Assume internal network `10.0.0.0/24`, external ips `1.2.3.0/24`, and the middleware VM running on `2.2.2.2`.
```
{
    "bwvisu-home-prefix" : "/shared/home",
    "gateway-nodes" : [
        {
            "internal-ip" : "10.0.0.1",
            "external-ip" : "1.2.3.1",
            "user" : "bwvisurunner",
            "port" : 22
        },
        {
            "internal-ip" : "10.0.0.2",
            "external-ip" : "1.2.3.2",
            "user" : "bwvisurunner",
            "port" : 22
        }

    ],
    "command-nodes" : [
        {
            "ip" : "1.2.3.1",
            "user" : "bwvisurunner",
            "port" : 22                                                                                                                                        
        },
        {
            "ip" : "1.2.3.2",
            "user" : "bwvisurunner",
            "port" : 22
        }
    ],
    "keyholed-servers" : [ { "ip": "2.2.2.2", "port" : 80} ],
    "keyholed-port-range" : { "min" : 50000, "max" : 60000 },
    "keyholed-max-forwarders-per-job" : 100,
    "etcd-servers" : [ { "ip": "127.0.0.1", "port" : 2379} ],
    "debug" : {
        "keyholed-static-forwarding" : "False"
    },
    "forwarding-hostname-suffix" : "-ib0"
}

```

Explanation of the fields:
* `bwvisu-home-prefix`: A directory such that `$bwvisu-home-prefix/$user-id` corresponds to a user-specific directory where bwVisu can deposit job logfiles. This directory should be read/writable by the user, and only the user. A typical choice of this directory is the home directory.
* `gateway-nodes`: A list of nodes that will be used as gateways, i.e. the visualization data stream will be channeled from the internal compute nodes through the gateway nodes to the user
   * `internal-ip`: IP of the gateway node in the internal network of the cluster
   * `external-ip`: Publicly reachable IP of the gateway node
   * `user`: Name of the user that the middleware will use to log into the gateway nodes to setup `iptables` port forwarding
   * `port`: ssh port that the middleware should use to login
* `command-nodes`: A list of nodes to be used to manage jobs; i.e. invoke `sbatch`, `squeue`, `scancel`
   * `ip`: IP of the command node. Must be reachable from the middleware.
   * `user`: Name of the user that the middleware will use for login
   * `port`: ssh port that the middleware should use to login
* `keyholed-servers`: List of IPs and ports of keyhole server deployments. Since keyhole is part of the middleware and most deployments will feature only one middleware deployments, this is usually the IP of the middleware machine. Must be reachable from the compute nodes.
* `keyholed-port-range`: Range of ports on compute and gateway nodes that the middleware is allowed to use for bwVisu.
* `keyholed-max-forwarders-per-job`: Maximum number of forwarded ports that a single job is allowed to request.
* `etcd-servers`: IP and port of the etcd database servers that store the middleware's data
* `debug`
   * `keyholed-static-forwarding`: If set to true, will always assign the same port to jobs for debugging purposes. Should never be enabled on actual deployments.
* `forwarding-hostname-suffix`: This will be appended to the port forwarding target's hostname. For example, a job is running on the hostname `visu4`. The middleware now attempts to create a port forwarding rule from `gateway0` to `visu4$-forwarding-hostname-suffix`. This can be useful if there are multiple networks and a suffix is used to distinguish which network interface on the host is meant. In the example configuration, `-ib0` is used to enforce that the Infiniband network is used to transport the bwVisu visualization data stream.

### bwVisu applications on the backend

In order to start the bwVisu application images on the backend, you need to have installed
* `singularity`
* `Xpra`
* the bwVisu Xorg singularity image

