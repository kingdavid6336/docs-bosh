# Configuring NTP for a deployment

!!! note
    If you are on Azure, you cannot change NTP configuration because it will read it
    from a local PTP device.  For more information on recommended azure configuration
    using chrony, see
    [here](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/time-sync#chrony).

## Configuring NTP servers on all deployments

To configure NTP servers for all VMs on all deployments,
add an ntp section to the agent env in your bosh director manifest.
For example:

```
instance_groups
- name: bosh
  ...
  properties:
    agent:
      env:
        bosh:
          ntp:
          - time1.google.com
          - time2.google.com
          - time3.google.com
          - time4.google.com
```

This will configure the agent to update your NTP configuration before
synchronizing its clock with the specified NTP servers.

## Configuring NTP servers in the director itself

The bosh director is often deployed using `bosh create-env`. 
For VMs deployed using `bosh create-env`,
ntp should be configured by adding an ntp section to the `resource_pools`
in the create-env manifest
For example:

```
resource_pools:
- env:
    bosh:
      ntp:
      - time1.google.com
      - time2.google.com
      - time3.google.com
      - time4.google.com
  name: vms
  network: default
```

This will have the same agent behavior as changing the `agent.env.bosh.ntp` in a director manifest.


At this point you should know enough to configure NTP on your VMs. The rest of this document is explaining details or exceptions.

## It is also possible to configure the agent env on an instance-group basis

If you wanted to configure the NTP servers specifically for one
instance group in a deployment you can do so using `env.bosh.ntp`
property on that instance group in its deployment manifest.

In order to completely disable NTP client configuration on one instance group, 
specify an empty list `env.bosh.ntp=[]` on the instance group, and also configure
an empty ntp list in the [bosh director default configuration of the cpi-release](https://github.com/cloudfoundry/bosh-deployment/blob/master/vsphere/cpi.yml), for
example [vsphere-cpi](https://github.com/cloudfoundry/bosh-vsphere-cpi-release/blob/34e66c47002c42e37970a02256f13be07a5d138f/jobs/vsphere_cpi/spec#L97-L101)

## Configuring NTP via CPI job properties is DEPRECATED

If you configure NTP on an instance group that has CPI jobs, some CPIs will pick
up that configuration and place it in the agent settings on deployed VMs. The
agent will detect those settings and use them as a fallback in the event that
the agent env does not include NTP configuration. This configuration style is
deprecated since it is not guaranteed to take precedence and the configuration
style may vary across IaaSes.
