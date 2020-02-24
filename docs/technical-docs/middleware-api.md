# bwVisu Middleware API

## General
* All messages must be of ContentType application/json
* Authentication presently works with HTTP basic auth (username bwvisu-web) over HTTPS
* Port is 4242

## Submitting a new job - POST to /jobs
* Expected JSON body (example):
```
{
"uid" : "1005",
"template-name" : "template_xpra.txt",
"variables" : {
"expected_runtime_in_minutes" : "15",
"number_of_nodes" : "1",
"tasks_per_node" : "1",
"image_name" : "ubuntu_nemo.simg"
}
}
```

* All variables within a template are sanitized - allowed characters are: 
   * Alphanumerical characters
   * `'.'`
   * `'/'`
   * `'-'`
   * `'_'`
* The template name is sanitized - forbidden characters are
   * `'/'`
   * `'\\'`
   * `'\n'`

### Individual parameters
* `uid`: The user id as string (user must exist on all cluster nodes)
* `template-name`: Filename of the template (sanitized as explained above)
* `variables`: Contains all variables to be substituted into the template
   * `expected_runtime_in_minutes`: String containing the maximum allowed runtime in minutes
   * `number_of_nodes`: Number of nodes as string
   * `tasks_per_node`: Number of MPI tasks per node as string
   * `image_name`: Name of the image - usual sanitation rules for variables apply
     The filename given here is relative to the home directory of the user.

### Execution and return value
* The job is always executed inside the home directory of the user. (Note: All job logs can be found in `$HOME` as `bwvisu-job-<jobid>.out`). 
Submission to slurm happens immediately (i.e. is done once the response from infectoid arrives at the web UI), but the actual execution
may be postponed by slurm.

* On success, HTTP 200 and a JSON message is returned containing the job id:
```
{
  'job-id': '<job id as string>'
}
```

## Cancelling a job - DELETE to /jobs/<job-id>

* On success, HTTP 200 and the following message is returned:
```
{'response' : 'job cancelled'}
```
* If the job does not exist, has already finished, or was not submitted by infectoid, HTTP 404 is returned with the detail message "no such running job"

## Querying a job - GET to /jobs/<job-id>
* If the job does not exist or was not submitted by infectoid,  HTTP 404 is returned with the detail message "no such job"
* On success, HTTP 200 and a JSON message similar to the following example is returned:
```
{
  'job-id' : '1234',
  'batch-script' : {
    'template-name' : 'template_xpra.txt',
    'variables' : {<lots of stuff>}
  },
  'services' : [{
    'internal-port' : '50000'
    'external-port' : '50000'
    'external-ip' : '129.206.9.65'
  }],
  'state' : 'available',
  'uid' : '1005',
  'details' : {<lots of stuff, all details that SLURM knows about this job}
}
```

### Individual fields
* `job-id`: The if of the job as string
* `batch-script`: The batch script template name and variables as they were provided at submission
* `state`: The state of the job as string:
   - `'submitted'`: The job has been submitted to the SLURM queue but has not yet started running
   - `'running'`: The job has started execution, but is not yet available for connecting (Note: presently, this state does not appear in practice)
   - `'available'`: The job is up and ready to be connected by the user
   - `'completing'`: The job is about to terminate. This state *may* be assumed by a job after being cancelled.
   - `'finished'`: The job has completed and is not running anymore
* `uid`: A string containing the user id of the user who has submitted this job
* `services`: Contains a list of exposed services. This list is only non-empty if the job is in the state 'available'.
  In that case, each element contains the following subkeys:
   - `internal-port`: The port (as integer) on the internal compute nodes that has been allocated to the job
   - `external-port`: The port (as integer) on the external node that is forwarded to the compute node.
   - `external-ip`: IP addresses where the external port is available
  (Implementation detail: If the job has been queried in the state 'available' before, these fields may also be present in the state 'finished' with their last known values)
* `details`: Contains everything that SLURM can tell us about the job. This field is only guaranteed to exist in the state 'running' or 'available'.
  (Implementation detail: If the job has been queried in the state 'available' or 'running' before, these fields may also be present in the state 'finished' with their last known values)
  
## List all running jobs - GET to /jobs
* Returns HTTP 200 and the following JSON message:
```
{ 'jobs' : [<running jobs>] }
```
where "running jobs" is a list of the job ids (as string) of all running jobs.

## Checking if a user exists on the cluster - GET to /cluster/check-user/<username>
In order to run a a job, it is necessary that a user exists on the cluster.
This can be queried (e.g. in order to show an error message as early as possible, or preventing login to bwvisu web entirely if the user doesn't exist) using the request in the title.
* If the user exists, HTTP 200 is returned with the body:
```
{'response' : 'success'}
```
* If the user does not exist, HTTP 404 is returned.
