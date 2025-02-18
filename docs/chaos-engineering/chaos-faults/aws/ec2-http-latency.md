---
id: ec2-http-latency
title: EC2 HTTP Latency
---

## Introduction

- EC2 HTTP latency has chaos to disrupt the state of infra resources. This experiment induces HTTP chaos on an AWS EC2 instance using the Amazon SSM Run Command, carried out using SSM Docs that is in-built in the experiment for the given chaos scenario.
- It injects HTTP response latency to the service whose port is specified using `TARGET_SERVICE_PORT` by starting the proxy server and redirecting the traffic through the proxy server.
- It introduces HTTP latency chaos on the EC2 instance using an SSM doc for a certain chaos duration.

:::tip Fault execution flow chart
![EC2 HTTP Latency](./static/images/ec2-http-latency.png)
:::

## Prerequisites

:::info

- Kubernetes >= 1.17

**AWS EC2 Access Requirements:**

- SSM agent is installed and running in the target EC2 instance.
- Kubernetes secret with AWS Access Key ID and Secret Access Key credentials in the `CHAOS_NAMESPACE`. A secret file looks like:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: cloud-secret
type: Opaque
stringData:
  cloud_config.yml: |-
    # Add the cloud AWS credentials respectively
    [default]
    aws_access_key_id = XXXXXXXXXXXXXXXXXXX
    aws_secret_access_key = XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
```

- If you change the secret, update the `experiment.yml` environment values to extract the relevant data from the secret. Also account for the path at which this secret is mounted as a file in the manifest environment variable `AWS_SHARED_CREDENTIALS_FILE`.

### NOTE

You can pass the VM credentials as secrets or as a chaosengine environment variable.
:::

## Default Validations

:::info

- The EC2 instance should be in a healthy state.

:::

## Experiment Tunables

<details>
    <summary>Check the Experiment Tunables</summary>
    <h2>Mandatory Fields</h2>
    <table>
        <tr>
            <th> Variables </th>
            <th> Description </th>
            <th> Notes </th>
        </tr>
        <tr>
          <td> EC2_INSTANCE_ID </td>
          <td> ID of the target EC2 instance </td>
          <td> For example: <code>i-044d3cb4b03b8af1f</code> </td>
        </tr>
        <tr>
          <td> REGION </td>
          <td> The AWS region ID where the EC2 instance has been created </td>
          <td> For example: <code>us-east-1</code> </td>
        </tr>
        <tr>
            <td> LATENCY </td>
            <td> Provide latency to be added to request in miliseconds.</td>
            <td> For example: 1000</td>
        </tr>
        <tr>
            <td> TARGET_SERVICE_PORT </td>
            <td> Port of the service to target </td>
            <td> Defaults to port 80 </td>
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
            <td> TOTAL_CHAOS_DURATION </td>
            <td> The total time duration for chaos insertion (in sec) </td>
            <td> Defaults to 30s </td>
        </tr>
        <tr>
            <td> CHAOS_INTERVAL </td>
            <td> The interval (in sec) between successive instance termination </td>
            <td> Defaults to 30s </td>
        </tr>
        <tr>
            <td> AWS_SHARED_CREDENTIALS_FILE </td>
            <td> Provide the path for AWS secret credentials</td>
            <td> Defaults to <code>/tmp/cloud_config.yml</code> </td>
          </tr>
        <tr>
            <td> SEQUENCE </td>
            <td> Defines the sequence of chaos execution for multiple instances </td>
            <td> Default value: parallel. Supported: serial, parallel </td>
        </tr>
        <tr>
            <td> RAMP_TIME </td>
            <td> Period to wait before and after injection of chaos (in sec) </td>
            <td> For example: 30 </td>
        </tr>
        <tr>
            <td> INSTALL_DEPENDENCY </td>
            <td> Whether to install the dependency to run the experiment </td>
            <td> If the dependency already exists, you can turn it off (defaults to True)</td>
        </tr>
        <tr>
            <td> PROXY_PORT  </td>
            <td> Port where the proxy listens for requests</td>
            <td> Defaults to 20000 </td>
        </tr>
        <tr>
            <td> TOXICITY </td>
            <td> Percentage of HTTP requests affected </td>
            <td> Defaults to 100 </td>
        </tr>
        <tr>
          <td> NETWORK_INTERFACE  </td>
          <td> Network interface used for the proxy</td>
          <td> Defaults to `eth0` </td>
        </tr>
    </table>
</details>

## Experiment Examples

### Common Experiment Tunables

Refer to the [common attributes](../common-tunables-for-all-experiments) to tune the common tunables for all the experiments.

### Target Service Port

It is the targeted service's port being targeted. You can tune it using the `TARGET_SERVICE_PORT` environment variable.

You can use the following example to tune it:

[embedmd]:# (./static/manifests/http-latency/target-service-port.yaml yaml)
```yaml
## provide the port of the targeted service
apiVersion: litmuschaos.io/v1alpha1
kind: ChaosEngine
metadata:
  name: engine-nginx
spec:
  engineState: "active"
  chaosServiceAccount: litmus-admin
  experiments:
  - name: ec2-http-latency
    spec:
      components:
        env:
        # provide the port of the targeted service
        - name: TARGET_SERVICE_PORT
          value: "80"
```

### Proxy Port

It is the port where the proxy server listens for requests. You can tune it using the `PROXY_PORT` environment variable.

You can use the following example to tune it:

[embedmd]:# (./static/manifests/http-latency/proxy-port.yaml yaml)
```yaml
# provide the port for proxy server
apiVersion: litmuschaos.io/v1alpha1
kind: ChaosEngine
metadata:
  name: engine-nginx
spec:
  engineState: "active"
  chaosServiceAccount: litmus-admin
  experiments:
  - name: ec2-http-latency
    spec:
      components:
        env:
        # provide the port for proxy server
        - name: PROXY_PORT
          value: '8080'
        # provide the port of the targeted service
        - name: TARGET_SERVICE_PORT
          value: "80"
```

### Latency

It is the latency value that is added to the http request. You can tune it using the `LATENCY` environment variable.

You can use the following example to tune it:

[embedmd]:# (./static/manifests/http-latency/latency.yaml yaml)
```yaml
## provide the latency value
apiVersion: litmuschaos.io/v1alpha1
kind: ChaosEngine
metadata:
  name: engine-nginx
spec:
  engineState: "active"
  chaosServiceAccount: litmus-admin
  experiments:
  - name: ec2-http-latency
    spec:
      components:
        env:
        # provide the latency value
        - name: LATENCY
          value: '2000'
        # provide the port of the targeted service
        - name: TARGET_SERVICE_PORT
          value: "80"
```

### Toxicity

It defines the toxicity value to be added to the http request. You can tune it using the `TOXICITY` environment variable.
Toxicity value defines the percentage of the total number of http requests that are affected.

You can use the following example to tune it:

[embedmd]:# (./static/manifests/http-latency/toxicity.yaml yaml)
```yaml
## provide the toxicity
apiVersion: litmuschaos.io/v1alpha1
kind: ChaosEngine
metadata:
  name: engine-nginx
spec:
  engineState: "active"
  chaosServiceAccount: litmus-admin
  experiments:
  - name: ec2-http-latency
    spec:
      components:
        env:
        # toxicity is the probability of the request to be affected
        # provide the percentage value in the range of 0-100
        # 0 means no request will be affected and 100 means all request will be affected
        - name: TOXICITY
          value: "100"
        # provide the port of the targeted service
        - name: TARGET_SERVICE_PORT
          value: "80"
```

### Network Interface

It defines the network interface used for the proxy. You can tune it using the `NETWORK_INTERFACE` environment variable.

You can use the following example to tune it:

[embedmd]:# (./static/manifests/http-latency/network-interface.yaml yaml)
```yaml
## provide the network interface for proxy
apiVersion: litmuschaos.io/v1alpha1
kind: ChaosEngine
metadata:
  name: engine-nginx
spec:
  engineState: "active"
  chaosServiceAccount: litmus-admin
  experiments:
  - name: ec2-http-latency
    spec:
      components:
        env:
        # provide the network interface for proxy
        - name: NETWORK_INTERFACE
          value: "eth0"
        # provide the port of the targeted service
        - name: TARGET_SERVICE_PORT
          value: '80'
```
