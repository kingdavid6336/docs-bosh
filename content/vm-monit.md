The Agent on each deployment job VM is responsible for managing lifecycle of each enabled release job. It starts, monitors, restarts and stops release jobs' processes. These tasks are done with the help of [Monit, version 5.2.5](https://web.archive.org/web/20110816041503/https://mmonit.com/monit/documentation/monit.html). The Agent communicates with the Monit daemon through Monit HTTP APIs to add, remove, start, stop, monitor and unmonitor release jobs' processes.
---
## Check Status {: #check-status }

Assuming you have a deployment, run `bosh instances` to see aggregate status for each deployment job VM:

```shell
bosh instances
```

Should result in:

```text

Deployment `my-deployment'

Director task 10326

Task 10326 done

+----------------+---------+---------------+-------------+
| Instance       | State   | Resource Pool | IPs         |
+----------------+---------+---------------+-------------+
| redis-master/0 | running | redis-servers | 10.10.30.71 |
| redis-slave/0  | running | redis-servers | 10.10.30.72 |
| redis-slave/1  | failing | redis-servers | 10.10.30.73 |
+----------------+---------+---------------+-------------+
```

There are several typical values for State:

- `running`: the Director received a response from the Agent and the Agent reported its aggregate status as successful. Running state indicates that all release jobs' processes are successfully running at that moment.

- `failing`: the Director received a response from the Agent and the Agent reported its aggregate status as not successful. Failing state indicates _one_ or more of the release jobs' processes is not successfully running (could be failing to start, or exiting after some time, or still in the process of becoming healthy, etc.).

- `starting`: the Director received a response from the Agent and the Agent reported its aggregate status as starting. Starting state indicates that _one_ or more of the release jobs' processes are being started and may not yet be running.

- `unresponsive`: the Director did not receive any response from the Agent

To determine what the problem is with a specific VM, you can ssh into the VM and look at the logs and/or Monit directly.

---
## Using Monit on the VM {: #using-monit }

On any BOSH-managed VM, you can access Monit status for release jobs'
processes via Monit CLI. Before you can run the `monit` command you have to
switch to become a `root` user (via `sudo su` or `sudo -i`) since Monit
executable is only available to root users.

Each enabled release job has its own directory in `/var/vcap/jobs/` directory. Each release job directory contains a monit file (e.g. `/var/vcap/jobs/redis-server/monit`) with final monit configuration for that release job. This is how you can tell which processes belong to which release job. Most release job only start a single process.

!!! note
    Monit configuration file in release job directory is just a copy of the actual Monit configuration. Changing it will not affect running Monit configuration.

To view status for all processes Monit is managing you can run `monit summary`:

```shell
monit summary
```

Should result in:

```text
The Monit daemon 5.2.4 uptime: 1d 22h 7m

Process 'nats'                      running
Process 'redis'                     running
Process 'postgres'                  running
Process 'blobstore_nginx'           running
Process 'director'                  running
Process 'worker_1'                  running
Process 'worker_2'                  running
Process 'worker_3'                  running
Process 'director_scheduler'        running
Process 'director_nginx'            running
Process 'registry'                  running
Process 'health_monitor'            running
System 'system_bm-24638eb6-55b9-4670-bb1a-23c9e3f77d91' running
```

!!! note
    You can use standard <code>watch</code> utility with the summary command to track process status over time.

You can also get more detailed information about individual processes via `monit status`:

```shell
monit status
```

Should result in:

```text
The Monit daemon 5.2.4 uptime: 1d 22h 8m

Process 'nats'
  status                            running
  monitoring status                 monitored
  pid                               2951
  parent pid                        1
  uptime                            1d 22h 8m
  children                          0
  memory kilobytes                  24420
  memory kilobytes total            24420
  memory percent                    0.6%
  memory percent total              0.6%
  cpu percent                       0.0%
  cpu percent total                 0.0%
  data collected                    Thu Dec  4 22:44:36 2014
...
```

While debugging why certain process is failing it is usually useful to tell Monit to stop restarting the failing process. You can do so via `monit stop <process-name>` command. To start it back up use `monit start <process-name>` command.

See the [Archived Monit Documentation, for version 5.2.5](https://web.archive.org/web/20110816041503/https://mmonit.com/monit/documentation/monit.html) to learn more about Monit.
