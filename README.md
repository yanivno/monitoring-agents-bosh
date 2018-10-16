# Create BOSH release
**Note:** When working on a BOSH release always work in the release root directory.

To create a development release (dirty git state) run `bosh create-release --force`
To create a final release run (clean git state) `bosh create-release`

After creating the release upload it using `bosh upload-release`

**TODO**: Find a way how to share the blobs. At the moment they are not uploaded and distributed via `local-blobstore`
```
$ bosh blobs
telegraf/telegraf-1.8.1_linux_amd64.tar.gz	13 MiB	(local)	b898f6cf0b3f990c75bcb775ac14189baad6d55d	
```

# Create runtime config

Create a runtime config based per PKS cluster. To specify the target cluster use the BOSH deployment name `service-instance_CLUSTERUUID`

```
releases:
- {name: monitoring-agents, version: 0+dev.14}

addons:
- name: monitoring-agents
  jobs:
  - name: telegraf
    release: monitoring-agents
    properties:
      influxdb_agent_interval: "10s"
      influxdb_telegraf:
      - url: "http://192.168.178.118:8086"
        db: "server1-db"
  include:
    deployments: [service-instance_a034c619-3395-44b7-9f92-2eb7928328bd]
```

In the above example the runtime config will only be applied the PKS cluster with the ID `a034c619-3395-44b7-9f92-2eb7928328bd`.

To extract the cluster UUID run `pks clusters`

```
Name       Plan Name  UUID                                  Status     Action
cluster01  small      a034c619-3395-44b7-9f92-2eb7928328bd  succeeded  CREATE
```

or `pks cluster cluster-name`

```
Name:                     cluster01
Plan Name:                small
UUID:                     a034c619-3395-44b7-9f92-2eb7928328bd
Last Action:              CREATE
Last Action State:        succeeded
Last Action Description:  Instance provisioning completed
Kubernetes Master Host:   cluster01.k8s.home.lab
Kubernetes Master Port:   8443
Worker Nodes:             3
Kubernetes Master IP(s):  192.168.130.9
Network Profile Name:     
```

# Deploy runtime config
Fetch named runtime config `bosh runtime-config --name='cluster01-monitoring'`
Update names runtime config `bosh update-runtime-config --name='cluster01-monitoring' manifest/runtime-config.yml`
