# UrbanCode Deploy Agent Relay - OpenShift Template

[UrbanCode Deploy Agent Relay](https://developer.ibm.com/urbancode/products/urbancode-deploy/) is a tool for automating application deployments through your environments. It is designed to facilitate rapid feedback and continuous delivery in agile development while providing the audit trails, versioning and approvals needed in production.

## Introduction

This chart deploys a single instance of the UrbanCode Deploy agent relay.

## Template Details
* Includes a StatefulSet workload object

## Prerequisites

1. The UrbanCode agent relay must have a UrbanCode Deploy server to connect to.

2. A PersistentVolume (PV) that will hold the conf directory for the UrbanCode Deploy agent relay is required.  This same PV is used to persist the agent relay cache data if caching is enabled and persisted.  If your cluster supports dynamic volume provisioning you will not need to create a PV or PersistentVolumeClaim (PVC) before installing this chart.  If your cluster does not support dynamic volume provisioning, you will need to either ensure a PV is available or you will need to create one before installing this chart.  You can optionally create the PVC to bind it to a specific PV, or you can let the chart create a PVC and bind to any available PV that meets the required size and storage class.  Sample YAML to create the PV and PVC are provided below.

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: ucdr-conf-vol
  labels:
    volume: ucdr-conf-vol
spec:
  capacity:
    storage: 10Mi
  accessModes:
    - ReadWriteOnce
  nfs:
    server: 192.168.1.17
    path: /volume1/k8/ucdr-conf
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: ucdr-conf-volc
spec:
  storageClassName: ""
  accessModes:
    - "ReadWriteOnce"
  resources:
    requests:
      storage: 10Mi
  selector:
    matchLabels:
      volume: ucdr-conf-vol
```

## Resources Required
Kubernetes 1.9

## Application deployment using the template

Use the following command to deploy the UCD Server agent relay to an OpenShift project:

```bash
$ oc process -f https://raw.githubusercontent.com/UrbanCode/UCDAgentRelay-OpenShift/master/ucdr_template.yaml --param-file values.txt | oc create -f -
```
The above command uses the values.txt file to supply parameter values for the application deployment.
Here is the example contents of a values.txt file:
```
APPLICATION_NAME=sample-agent-relay
IMAGE_REPOSITORY=sample-registry.com/prod/ucdr
IMAGE_TAG=7.0.2.2.1017795
IMAGE_PULL_POLICY=Always
IMAGE_PULL_SECRET=sample-secret
JMS_PROXY_SERVER=myUCDServer:30613
```

The [configuration](#Configuration) section lists the parameters that can be set during installation.


## Configuration

### Parameters

The template uses the following parameter values that can be supplied in a separate parameters file.

##### Common Parameters

| Parameter  | Definition | Allowed Value |
|---|---|---|
| APPLICATION_NAME  | The name of the deployed application | |
| IMAGE_PULL_POLICY | Image Pull Policy | Always, Never, or IfNotPresent. Defaults to Always if :latest tag is specified, or IfNotPresent otherwise  |
| IMAGE_REPOSITORY | Name of image, including repository prefix (if required) | See [Extended description of Docker tags](https://docs.docker.com/engine/reference/commandline/tag/#extended-description) |
| IMAGE_TAG | Docker image tag | See [Docker tag description](https://docs.docker.com/engine/reference/commandline/tag/) |
| IMAGE_PULL_SECRET |  An image pull secret used to authenticate with the image registry | Empty (default) if no authentication is required to access the image registry. |
| EXISTING_CLAIM_NAME | The name of an existing Persistent Volume Claim that references the Persistent Volume that will be used to hold the UCD agent conf directory. |  |
| JMS_PROXY_SERVER | UrbanCode Deploy Server hostname and JMS port in the form hostname:port.  If specifying failover info, separate multiple hostname:port with a comma.  For example, ucd1.example.com:7918,ucd2.example.com:7918 |  |
| CODESTATION_ENABLE_REPLICATION | Specify true to enable artifact caching on the relay. | |
| CODESTATION_PERSIST_CACHE | Specify true to persist the artifact cache when the relay container is restarted. |  |
| CODESTATION_MAX_CACHE_SIZE | The size to which to limit the artifact cache, such as 500M for 500 MB or 5G for 5 GB.  To not put a limit on the cache, specify none. | |
| CODESTATION_SERVER_URL | The full URL of the central UCD Server to connect to, such as https://myserver.example.com:8443 |  |
| CODESTATION_SERVER_PASSWORD | An authentication token created by the UCD Server |  |
| CODESTATION_GEOTAGS | If you choose to cache files on the relay, you can specify one or more component version statuses here, separated by semicolons. The agent relay automatically caches component versions with any of these statuses so that those versions are ready when they are needed for a deployment. A status can contain a space except in the first or last position. A status can contain commas. The special * status replicates all artifacts, but use this status with caution, because it can make the agent relay store a large amount of data. If no value is specified, no component versions are cached automatically. |  |
| PERSISTENT_VOLUME_SIZE | Size of storage needed to persist Agent Relay configuration data and cache (if necessary).  The default is 10Mi. |  |

## Storage
See the Prerequisites section of this page for storage information.

## Limitations


