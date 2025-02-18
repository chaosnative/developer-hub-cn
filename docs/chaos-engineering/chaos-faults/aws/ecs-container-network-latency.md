---
id: ecs-container-network-latency
title: ECS Container Network Latency
---

## Introduction

- ECS Container Network Latency contains chaos to disrupt the state of infra resources. The experiment can induce chaos on AWS ECS container using Amazon SSM Run Command, this is carried out by using SSM Docs which is in-built in the experiment for the give chaos scenario.

- It causes network chaos on containers of ECS task with given cluster name using an SSM docs for a certain chaos duration.

- To select the Task Under Chaos (TUC) you can make use of servie name associated with the task that is, if you provide the service name along with cluster name only all the tasks associated with the given service will be selected as chaos targets.

- It tests the ECS task sanity (service availability) and recovery of the task containers subjected to network chaos.

:::tip Fault execution flow chart
![ECS Container Network Latency](./static/images/ecs-network-chaos.png)
:::

## Uses

<details>
<summary>View the uses of the experiment</summary>
<div>
The experiment causes network degradation of the task container without the container being marked unhealthy/unworthy of traffic from outside. The idea of this experiment is to simulate issues within your ECS task network OR communication across services in different availability zones/regions etc.

Mitigation (in this case keep the timeout i.e., network latency value low) could be via some middleware that can switch traffic based on some SLOs/perf parameters. If such an arrangement is not available the next best thing would be to verify if such a degradation is highlighted via notification/alerts etc,. so the admin/SRE has the opportunity to investigate and fix things. Another utility of the test would be to see what the extent of impact caused to the end-user OR the last point in the app stack on account of degradation in access to a downstream/dependent microservice. Whether it is acceptable OR breaks the system to an unacceptable degree. The experiment provides NETWORK_LATENCY so that you can control the chaos against specific services within or outside the cluster.

The task may stall or get corrupted while they wait endlessly for a packet. The experiment limits the impact (blast radius) to only the traffic you want to test by specifying service to find TUC (Task Under Chaos). This experiment will help to improve the resilience of your services over time.
</div>
</details>

## Prerequisites

:::info

- Ensure that Kubernetes Version >= 1.17

**AWS EC2 Access Requirement:**

- Ensure that you have sufficient AWS access to stop and start an EC2 instance.
- Ensure to create a Kubernetes secret having the AWS access configuration(key) in the `CHAOS_NAMESPACE`. A sample secret file looks like:

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
    aws_secret_access_key = XXXXXXXXXXXXXXX
```

- If you change the secret key name (from `cloud_config.yml`) please also update the `AWS_SHARED_CREDENTIALS_FILE` ENV value on `experiment.yaml`with the same name.

## Default Validations

:::info

- EC2 instance should be in healthy state.

:::

## Experiment tunables

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
        <td> CLUSTER_NAME </td>
        <td> Name of the target ECS cluster</td>
        <td> Eg. cluster-1 </td>
        </tr>
        <tr>
        <td> REGION </td>
        <td> The region name of the target ECS cluster</td>
        <td> Eg. us-east-1 </td>
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
        <td> The total time duration for chaos insertion (sec) </td>
        <td> Defaults to 30s </td>
      </tr>
      <tr>
        <td> CHAOS_INTERVAL </td>
        <td> The interval (in sec) between successive instance termination.</td>
        <td> Defaults to 30s </td>
      </tr>
      <tr> 
        <td> AWS_SHARED_CREDENTIALS_FILE </td>
        <td> Provide the path for aws secret credentials</td>
        <td> Defaults to <code>/tmp/cloud_config.yml</code> </td>
      </tr>
      <tr> 
        <td>  NETWORK_LATENCY </td>
        <td> Provide the value of latency in ms	</td>
        <td> Defaults to 2000 </td>
      </tr>
      <tr> 
          <td> DESTINATION_IPS </td>
          <td> IP addresses of the services or the CIDR blocks(range of IPs), the accessibility to which is impacted </td>
          <td> Comma separated IP(S) or CIDR(S) can be provided. if not provided, it will induce network chaos for all ips/destinations </td>
      </tr>
      <tr> 
          <td> DESTINATION_HOSTS </td>
          <td> DNS Names of the services, the accessibility to which, is impacted </td>
          <td> if not provided, it will induce network chaos for all ips/destinations or DESTINATION_IPS if already defined </td>
      </tr>
      <tr>
          <td> NETWORK_INTERFACE  </td>
          <td> Name of ethernet interface considered for shaping traffic </td>
          <td> Defaults to <code>eth0</code> </td>
      </tr>
      <tr> 
        <td> JITTER </td>
        <td> Provide the value of Iitter	</td>
        <td> Defaults to 0 </td>
      </tr>
      <tr>
        <td> SEQUENCE </td>
        <td> It defines sequence of chaos execution for multiple instance</td>
        <td> Default value: parallel. Supported: serial, parallel </td>
      </tr>
      <tr>
        <td> RAMP_TIME </td>
        <td> Period to wait before and after injection of chaos in sec </td>
        <td> Eg. 30 </td>
      </tr>
    </table>
</details>

## Experiment Examples

### Common and AWS specific tunables

Refer the [common attributes](../common-tunables-for-all-experiments) and [AWS specific tunable](./aws-experiments-tunables) to tune the common tunables for all experiments and aws specific tunables.

### Network Latency

It defines the network latency(in ms) to be injected in the targeted container. It can be tuned via `NETWORK_LATENCY` ENV.

Use the following example to tune this:

[embedmd]:# (./static/manifests/ecs-network-chaos/network-latency.yaml yaml)
```yaml
# injects network latency for a certain chaos duration
apiVersion: litmuschaos.io/v1alpha1
kind: ChaosEngine
metadata:
  name: engine-nginx
spec:
  engineState: "active"
  annotationCheck: "false"
  chaosServiceAccount: litmus-admin
  experiments:
  - name: ecs-container-network-latency
    spec:
      components:
        env:
        # network latency to be injected
        - name: NETWORK_LATENCY
          value: '2000' #in ms
        - name: TOTAL_CHAOS_DURATION
          value: '60'
```

### Network Interface

The defined name of the ethernet interface, which is considered for shaping traffic. It can be tuned via `NETWORK_INTERFACE` ENV. Its default value is `eth0`.

Use the following example to tune this:

[embedmd]:# (./static/manifests/ecs-network-chaos/latency-network-interface.yaml yaml)
```yaml
# provide the network interface
apiVersion: litmuschaos.io/v1alpha1
kind: ChaosEngine
metadata:
  name: engine-nginx
spec:
  engineState: "active"
  annotationCheck: "false"
  chaosServiceAccount: litmus-admin
  experiments:
  - name: ecs-container-network-latency
    spec:
      components:
        env:
        # name of the network interface
        - name: NETWORK_INTERFACE
          value: 'eth0'
        - name: TOTAL_CHAOS_DURATION
          value: '60'
```

### Jitter

It defines the jitter (in ms), a parameter that allows introducing a network delay variation. It can be tuned via `JITTER` ENV. Its default value is `0`.
Use the following example to tune this:

[embedmd]:# (./static/manifests/ecs-network-chaos/jitter.yaml yaml)
```yaml
# provide the network latency jitter
apiVersion: litmuschaos.io/v1alpha1
kind: ChaosEngine
metadata:
  name: engine-nginx
spec:
  engineState: "active"
  annotationCheck: "false"
  chaosServiceAccount: litmus-admin
  experiments:
  - name: ecs-container-network-latency
    spec:
      components:
        env:
        # value of the network latency jitter (in ms)
        - name: JITTER
          value: '200'
```

### Destination IPs And Destination Hosts

The network experiments interrupt traffic for all the IPs/hosts by default. The interruption of specific IPs/Hosts can be tuned via `DESTINATION_IPS` and `DESTINATION_HOSTS` ENV.

- `DESTINATION_IPS`: It contains the IP addresses of the services or pods or the CIDR blocks(range of IPs), the accessibility to which is impacted.
- `DESTINATION_HOSTS`: It contains the DNS Names/FQDN names of the services, the accessibility to which, is impacted.

Use the following example to tune this:

[embedmd]:# (./static/manifests/ecs-network-chaos/latency-destination-ip-and-hosts.yaml yaml)
```yaml
# it inject the chaos for the egress traffic for specific ips/hosts
apiVersion: litmuschaos.io/v1alpha1
kind: ChaosEngine
metadata:
  name: engine-nginx
spec:
  engineState: "active"
  annotationCheck: "false"
  chaosServiceAccount: litmus-admin
  experiments:
  - name: ecs-container-network-latency
    spec:
      components:
        env:
        # supports comma separated destination ips
        - name: DESTINATION_IPS
          value: '8.8.8.8,192.168.5.6'
        # supports comma separated destination hosts
        - name: DESTINATION_HOSTS
          value: 'nginx.default.svc.cluster.local,google.com'
        - name: TOTAL_CHAOS_DURATION
          value: '60'
```

