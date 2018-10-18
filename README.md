# monitoring-agents-boshrelease
BOSH release that installs monitoring agents like
- Telegraf
- Appdynamicas Machine Agent

## Creating a development bosh release
```
bosh create-release --force
bosh upload-release
```

### Deploying to a Bosh Director
Sample deployment that deploys the monitoring agents on a instance group as well a as a runtime config can be found in the `manifest` directory.

```
bosh -d monitoring-agents-test deploy manifest/monitoring-agents.xml
```

## Creating a final BOSH release

1. If new blobs were uploaded upload them using `bosh upload-blobs` and `git commit` the changes.
 
1. create bosh final release (requires s3 credentials in `config/final.yml`)
```
export VERSION=x
bosh create-release --final --version=$VERSION --tarball=releases/monitoring-agents/-$VERSION.tgz
```

1. commit and push changes
```
git add releases .final_builds/
git commit -m"Release $VERSION"
git tag v$VERSION
git push origin master
git push --tags
```

1. Create a release from the new tag and upload the tarball `releases/monitoring-agents/monitoring-agents-$VERSION.tgz`.

It is helpful to include a `releases` snippet showing how to include this release into a deployment. To generate the sha of the tarball blob:
```
shasum releases/monitoring-agents/monitoring-agents-$VERSION.tgz
```

## Create runtime config

Create a runtime config based per PKS cluster. To specify the target cluster use the BOSH deployment name `service-instance_CLUSTERUUID`

```
releases:
- {name: monitoring-agents, version: 1}

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
