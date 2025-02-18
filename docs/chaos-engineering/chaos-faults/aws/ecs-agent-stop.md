---
id: ecs-agent-stop
title: ECS Agent Stop
---

## Introduction

- ECS Agent Stop contains chaos to disrupt the state of infra resources. The experiment can induce an agent stop chaos on AWS ECS using Amazon SSM Run Command, this is carried out by using SSM Docs which is in-built in the experiment for the give chaos scenario.

- It causes agent container stop on ECS with a given `CLUSTER_NAME` using an SSM docs for a certain chaos duration.

:::tip Fault execution flow chart
![ECS Agent Stop](./static/images/ecs-agent-stop.png)
:::

## Uses

<details>
<summary>View the uses of the experiment</summary>
<div>
Agent chaos stop is another very common and frequent scenario we find with ECS clusters that can break an agent that manages the task container on the ECS cluster and impacts their delivery. Such scenarios can still occur despite whatever availability aids docker provides.

Killing the agent container will distrupt the performance of it and impact to smooth working of task containers. So this category of chaos experiment helps to build the immunity on the application undergoing any such scenarios.

</div>
</details>

## Prerequisites

:::info

- Ensure that Kubernetes Version >= 1.17

**AWS EC2 Access Requirement:**

- Ensure that the ECS container metadata is enabled this feature is disabled by default. To enable it please follow the aws docs to [Enabling container metadata](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/container-metadata.html). If you have your task running prior this activity you may need to restart it to get the metadata directory as mentioned in the docs.

- Ensure that both you and ECS cluster instances have a Role with required AWS access to do SSM and ECS operations. Refer the below mentioned sample policy for the experiment (please note that the sample policy can be minimised further). To know more checkout [Systems Manager Docs](https://docs.aws.amazon.com/systems-manager/latest/userguide/setup-launch-managed-instance.html). Also, please refer the below mentioned policy required for the experiment.

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

- ECS container instance should be in healthy state.

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
          <td> Single name supported eg: <code>demo-cluster</code></td>
        </tr>
        <tr>
          <td> REGION </td>
          <td> The AWS region name of the target ECS cluster</td>
          <td> Eg: <code>us-east-2</code></td>
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

### Agent Stop

It derives the target agent from `CLUSTER_NAME` and stop them for a certain chaos duration. 

Use the following example to tune this:

[embedmd]:# (./static/manifests/ecs-agent-stop/agent-stop.yaml yaml)
```yaml
# stops the agent of an ecs cluster
apiVersion: litmuschaos.io/v1alpha1
kind: ChaosEngine
metadata:
  name: engine-nginx
spec:
  engineState: "active"
  annotationCheck: "false"
  chaosServiceAccount: litmus-admin
  experiments:
  - name: ecs-agent-stop
    spec:
      components:
        env:
        # provide the name of ecs cluster
        - name: CLUSTER_NAME
          value: 'demo'
        - name: REGION
          value: 'us-east-2'
        - name: TOTAL_CHAOS_DURATION
          VALUE: '60'
```
