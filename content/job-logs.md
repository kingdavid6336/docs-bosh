This topic describes different types of logs and how to access them.

## VM logs {: #vm-logs }

You can access logs from any VM:

* via [`bosh ssh` command](sysadmin-commands.md#ssh) to SSH into a VM and look at the log files
* via [`bosh logs` command](sysadmin-commands.md#logs) to download logs from the VM

The following sections describe different types of logs found on each BOSH managed VM.

---
### Job logs {: #job-logs }

Release jobs on VMs produce logs throughout different lifecycle events. Release authors are strongly encouraged to place release job logs into `/var/vcap/sys/log/<release_job_name>/*.log`, providing a consistent place for the operator to find them.

For example `redis-server` release job will create two log files:

- `/var/vcap/sys/log/redis-server/redis-server.stdout.log`
- `/var/vcap/sys/log/redis-server/redis-server.stderr.log`

To download logs from all release jobs on a specific VM, run `bosh logs <job_name> <index>`.

See additional information about following job lifecycle events' logs:

- [pre-start script logs](pre-start.md#logs)
- [drain script logs](drain.md#logs)

---
### Errand logs {: #errand-logs }

Unlike regular job logs BOSH does not automatically redirect errand logs to `/var/vcap/sys/log/*` directory, though we are planning to do so in future.

Errand's stdout and stderr output will be shown by the CLI when it's smaller than 1MB. If you expect errand to generate output larger than 1MB, currently it needs to be redirected to a file (by convention to `/var/vcap/sys/log/<job_name>/stdout.log`) from the errand script and then downloaded, or error will be returned upon errand completion.

To save output from an errand VM:

1. In the errand run script, redirect the output to a log.
1. Using the CLI, run `bosh run-errand X` with the `--download-logs` option to download the logs.

    By default, the CLI downloads the logs to your present working directory. Use the `--logs-dir destination_directory` option to change this directory.

```shell
bosh run-errand smoke-tests --download-logs --logs-dir ~/workspace/smoke-tests-logs
```

!!! note
    By default upon errand completion errand VM is deleted, so you cannot access logs saved to disk by the errand. You can use <code>--keep-alive</code> flag when running an errand to keep the VM with its logs.

---
### Monit logs {: #monit-logs }

The Agent uses Monit to start, restart, and stop release job processes as specified by the release jobs. Monit detects errors and outputs often useful information to its log. Use `tail` to examine the `monit.log` on a VM:

```shell
sudo tail -f -n 200 /var/vcap/monit/monit.log
```

---
### Agent logs {: #agent-logs }

Agent logs contain configuration and runtime information from the Agent running on a VM. Review these logs if the Director sees VM as unresponsive or the Director fails to contact it during its creation.

The Agent stores logs in `/var/vcap/bosh/log/` and outputs most recent content to `/var/vcap/bosh/log/current`.

```shell
sudo tail -f -n 200 /var/vcap/bosh/log/current
```

!!! note
    Agent logs are only accessible to the root user.

---
### System logs {: #system-logs }

System logs contain configuration and runtime information from the Linux kernel and other process running on a VM that are not directly managed by the BOSH Agent. These logs are stored in `/var/log` and are occasionally of interest when debugging OS-level problems, or when determining whether or not a VM is undersized for its workload. `auditd` and `sar` logs are also stored here.

If you're a Linux system administrator, you already know exactly the sorts of things that are in here -- BOSH does nothing particularly special with these logs.

!!! Note
    System logs are generally only accessible to the root user.

---
### Log rotation {: #log-rotation }

BOSH rotates release job logs with the [Logrotate][logrotate] log file management utility. Logrotate is configured by the Agent to act on all `.log` files in the `/var/vcap/sys/log/`, `/var/vcap/sys/log/*/`, and `/var/vcap/sys/log/*/*/` directories.

See also the [“Global Configuration” section](vm-config.md#global) for more
details on the contract for log files.

Logs are rotated every 15 minutes (see [agent's `etcLogrotateDTemplate` configuration][etcLogrotateDTemplate] for detailed settings).

[logrotate]: http://linuxconfig.org/logrotate
[etcLogrotateDTemplate]: https://github.com/cloudfoundry/bosh-agent/blob/f0b849f/platform/linux_platform.go#L594

---
### Syslog configuration {: #syslog-conf }

Recommended way to configure syslog forwarding on all or some VMs is to use [`syslog_forwarder` job from `syslog-release` as an addon](addons-common.md#syslog).

---
## Director task logs {: #director-logs }

When you run a [CLI](bosh-cli.md) command, the Director stores all activities for the specific command in a task log. Review these logs when you experience an issue with a command.

To access Director task logs:

1. Run [`bosh tasks recent`](sysadmin-commands.md#tasks) to find the task number of the command.
1. Run [`bosh task <task_number>`](sysadmin-commands.md#tasks).
