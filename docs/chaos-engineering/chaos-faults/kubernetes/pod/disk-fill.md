---
id: disk-fill
title: Disk Fill
---
## Introduction
- It causes Disk Stress by filling up the ephemeral storage of the pod on any given node.
- It causes the application pod to get evicted if the capacity filled exceeds the pod's ephemeral storage limit.
- It tests the Ephemeral Storage Limits, to ensure those parameters are sufficient.
- It tests the application's resiliency to disk stress/replica evictions.

:::tip Fault execution flow chart
![Disk Fill](./static/images/disk-fill.png)
:::

## Uses
<details>
<summary>View the uses of the experiment</summary>
<div>
Coming soon.
</div>
</details>

## Prerequisites
:::info
- Ensure that Kubernetes Version > 1.16.
- Appropriate Ephemeral Storage Requests and Limits should be set for the application before running the experiment. An example specification is shown below:
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: frontend
    spec:
      containers:
      - name: db
        image: mysql
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: "password"
        resources:
          requests:
            ephemeral-storage: "2Gi"
          limits:
            ephemeral-storage: "4Gi"
      - name: wp
        image: wordpress
        resources:
          requests:
            ephemeral-storage: "2Gi"
          limits:
            ephemeral-storage: "4Gi"
    ```
:::

## Default Validations
:::note
The application pods should be in running state before and after chaos injection.
:::

## Experiment tunables
<details>
    <summary>Check the Experiment Tunables</summary>
    <h2>Optional Fields</h2>
    <table>
      <tr>
        <th> Variables </th>
        <th> Description </th>
        <th> Notes </th>
      </tr>
      <tr> 
        <td> FILL_PERCENTAGE </td>
        <td> Percentage to fill the Ephemeral storage limit </td>
        <td> Can be set to more than 100 also, to force evict the pod. The ephemeral-storage limits must be set in targeted pod to use this ENV.</td>
      </tr>
      <tr>
        <td> EPHEMERAL_STORAGE_MEBIBYTES </td>
        <td> Ephemeral storage which need to fill (unit: MiBi)</td>
        <td>It is mutually exclusive with the <code>FILL_PERCENTAGE</code> ENV. If both are provided then it will use the <code>FILL_PERCENTAGE</code></td>
      </tr>
    </table>
    <h2>Optional Fields</h2>
    <table>
      <tr>
        <th> Variables </th>
        <th> Description </th>
        <th> Notes </th>
      </tr>
      <tr> 
        <td> TARGET_CONTAINER </td>
        <td> Name of container which is subjected to disk-fill </td>
        <td> If not provided, the first container in the targeted pod will be subject to chaos </td>
      </tr>
      <tr> 
        <td> CONTAINER_PATH </td>
        <td> Storage Location of containers</td>
        <td> Defaults to '/var/lib/docker/containers' </td>
      </tr>
      <tr> 
        <td> TOTAL_CHAOS_DURATION </td>
        <td> The time duration for chaos insertion (sec) </td>
        <td> Defaults to 60s </td>
      </tr>
      <tr>
        <td> TARGET_PODS </td>
        <td> Comma separated list of application pod name subjected to disk fill chaos</td>
        <td> If not provided, it will select target pods randomly based on provided appLabels</td>
      </tr> 
      <tr>
        <td> DATA_BLOCK_SIZE </td>
        <td> It contains data block size used to fill the disk(in KB)</td>
        <td> Defaults to 256, it supports unit as KB only</td>
      </tr> 
      <tr>
        <td> PODS_AFFECTED_PERC </td>
        <td> The Percentage of total pods to target  </td>
        <td> Defaults to 0 (corresponds to 1 replica), provide numeric value only </td>
      </tr> 
      <tr>
        <td> LIB </td>
        <td> The chaos lib used to inject the chaos </td>
        <td> Defaults to `litmus` supported litmus only </td>
      </tr>
      <tr>
        <td> LIB_IMAGE </td>
        <td> The image used to fill the disk </td>
        <td> Defaults to <code>litmuschaos/go-runner:latest</code> </td>
      </tr>
      <tr>
        <td> RAMP_TIME </td>
        <td> Period to wait before injection of chaos in sec </td>
        <td> Eg. 30 </td>
      </tr>
      <tr>
        <td> SEQUENCE </td>
        <td> It defines sequence of chaos execution for multiple target pods </td>
        <td> Default value: parallel. Supported: serial, parallel </td>
      </tr>
    </table>
</details>

## Experiment Examples

### Common and Pod specific tunables
Refer the [common attributes](../../common-tunables-for-all-experiments) and [Pod specific tunable](./common-tunables-for-pod-experiments) to tune the common tunables for all experiments and pod specific tunables. 

### Disk Fill Percentage

It fills the `FILL_PERCENTAGE` percentage of the ephemeral-storage limit specified at `resource.limits.ephemeral-storage` inside the target application. 

Use the following example to tune this:

[embedmd]:# (./static/manifests/disk-fill/fill-percentage.yaml yaml)
```yaml
## percentage of ephemeral storage limit specified at `resource.limits.ephemeral-storage` inside target application 
apiVersion: litmuschaos.io/v1alpha1
kind: ChaosEngine
metadata:
  name: engine-nginx
spec:
  engineState: "active"
  annotationCheck: "false"
  appinfo:
    appns: "default"
    applabel: "app=nginx"
    appkind: "deployment"
  chaosServiceAccount: litmus-admin
  experiments:
  - name: disk-fill
    spec:
      components:
        env:
        ## percentage of ephemeral storage limit, which needs to be filled
        - name: FILL_PERCENTAGE
          value: '80' # in percentage
        - name: TOTAL_CHAOS_DURATION
          VALUE: '60'
```

### Disk Fill Mebibytes

It fills the `EPHEMERAL_STORAGE_MEBIBYTES` MiBi of ephemeral storage of the targeted pod. 
It is mutually exclusive with the `FILL_PERCENTAGE` ENV. If `FILL_PERCENTAGE` ENV is set then it will use the percentage for the fill otherwise, it will fill the ephemeral storage based on `EPHEMERAL_STORAGE_MEBIBYTES` ENV.

Use the following example to tune this:

[embedmd]:# (./static/manifests/disk-fill/ephemeral-storage-mebibytes.yaml yaml)
```yaml
# ephemeral storage which needs to fill in will application
# if ephemeral-storage limits is not specified inside target application
apiVersion: litmuschaos.io/v1alpha1
kind: ChaosEngine
metadata:
  name: engine-nginx
spec:
  engineState: "active"
  annotationCheck: "false"
  appinfo:
    appns: "default"
    applabel: "app=nginx"
    appkind: "deployment"
  chaosServiceAccount: litmus-admin
  experiments:
  - name: disk-fill
    spec:
      components:
        env:
        ## ephemeral storage size, which needs to be filled
        - name: EPHEMERAL_STORAGE_MEBIBYTES
          value: '256' #in MiBi
        - name: TOTAL_CHAOS_DURATION
          VALUE: '60'
```

### Data Block Size

It defines the size of the data block used to fill the ephemeral storage of the targeted pod. It can be tuned via `DATA_BLOCK_SIZE` ENV. Its unit is `KB`.
The default value of `DATA_BLOCK_SIZE` is `256`.

Use the following example to tune this:

[embedmd]:# (./static/manifests/disk-fill/data-block-size.yaml yaml)
```yaml
# size of the data block used to fill the disk
apiVersion: litmuschaos.io/v1alpha1
kind: ChaosEngine
metadata:
  name: engine-nginx
spec:
  engineState: "active"
  annotationCheck: "false"
  appinfo:
    appns: "default"
    applabel: "app=nginx"
    appkind: "deployment"
  chaosServiceAccount: litmus-admin
  experiments:
  - name: disk-fill
    spec:
      components:
        env:
        ## size of data block used to fill the disk
        - name: DATA_BLOCK_SIZE
          value: '256' #in KB
        - name: TOTAL_CHAOS_DURATION
          VALUE: '60'
```

### Container Path

It defines the storage location of the containers inside the host(node/VM). It can be tuned via `CONTAINER_PATH` ENV. 

Use the following example to tune this:

[embedmd]:# (./static/manifests/disk-fill/container-path.yaml yaml)
```yaml
# path inside node/vm where containers are present 
apiVersion: litmuschaos.io/v1alpha1
kind: ChaosEngine
metadata:
  name: engine-nginx
spec:
  engineState: "active"
  annotationCheck: "false"
  appinfo:
    appns: "default"
    applabel: "app=nginx"
    appkind: "deployment"
  chaosServiceAccount: litmus-admin
  experiments:
  - name: disk-fill
    spec:
      components:
        env:
        # storage location of the containers
        - name: CONTAINER_PATH
          value: '/var/lib/docker/containers'
        - name: TOTAL_CHAOS_DURATION
          VALUE: '60'
```
