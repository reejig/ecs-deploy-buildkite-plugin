# ECS Deploy Buildkite Plugin [![Build status](https://badge.buildkite.com/02dd9bd7d4b4a6f3d80c198d7307e24bff9ae7e39ff1854bed.svg?branch=master)](https://buildkite.com/buildkite/plugins-ecs-deploy)

A [Buildkite plugin](https://buildkite.com/docs/agent/v3/plugins) for deploying to [Amazon ECS](https://aws.amazon.com/ecs/).

* Requires the aws cli tool be installed
* Registers a new task definition based on a given JSON file ([`register-task-definition`](http://docs.aws.amazon.com/cli/latest/reference/ecs/register-task-definition.html]))
* Updates the ECS service to use the new task definition ([`update-service`](http://docs.aws.amazon.com/cli/latest/reference/ecs/update-service.html))
* Waits for the service to stabilize ([`wait services-stable`](http://docs.aws.amazon.com/cli/latest/reference/ecs/wait/services-stable.html))

## Example

```yml
steps:
  - label: ":ecs: :rocket:"
    concurrency_group: "my-service-deploy"
    concurrency: 1
    plugins:
      - ecs-deploy#v1.4.1:
          cluster: "my-ecs-cluster"
          service: "my-service"
          task-definition: "examples/hello-world.json"
          task-family: "hello-world"
          image: "${ECR_REPOSITORY}/hello-world:${BUILDKITE_BUILD_NUMBER}"
```

## Options

### `cluster`

The name of the ECS cluster.

Example: `"my-cluster"`

### `service`

The name of the ECS service.

Example: `"my-service"`

### `task-definition`

The file path to the ECS task definition JSON file.

Example: `"ecs/task.json"`

### `task-family`

The name of the task family.

Example: `"my-task"`

### `image`

The Docker image to deploy. This can be an array to substitute multiple images in a single container definition.

Examples:
`"012345.dkr.ecr.us-east-1.amazonaws.com/my-service:123"`

```yaml
image:
  - "012345.dkr.ecr.us-east-1.amazonaws.com/my-service:123"
  - "012345.dkr.ecr.us-east-1.amazonaws.com/nginx:123"
```

### `task-role-arn` (optional)

An IAM ECS Task Role to assign to tasks.
Requires the `iam:PassRole` permission for the ARN specified.

### `target-group` (optional)

The Target Group ARN to map the service to.

Example: `"arn:aws:elasticloadbalancing:us-east-1:012345678910:targetgroup/alb/e987e1234cd12abc"`

### `target-container-name` (optional)

The Container Name to forward ALB requests to.

### `target-container-port` (optional)

The Container Port to forward requests to.

### `execution-role` (optional)

The Execution Role ARN used by ECS to pull container images and secrets.

Example: `"arn:aws:iam::012345678910:role/execution-role"`

Requires the `iam:PassRole` permission for the execution role.

### `deployment-configuration` (optional)

The minimum and maximum percentage of tasks that should be maintained during a deployment. Defaults to `100/200`

Example: `"0/100"`

### `region` (optional)

The region we deploy the ECS Service to.

### `requires-compatibilities` (optional)

The task launch type that Amazon ECS should validate the task definition against. This ensures that the task definition parameters are compatible with the specified launch type. If no value is specified, it defaults to EC2 .
Valid values include EC2 and FARGATE.

Example: `"FARGATE"`

### `task-cpu` (required if requires-compatibilities="FARGATE")

The number of CPU units used by the task. It can be expressed as an integer using CPU units, for example `1024`, or as a string using vCPUs, for example `1 vCPU` or `1 vcpu`.

Example: `1024`, `"1 vCPU"`, `"1 vcpu"`

### `task-memory` (required if requires-compatibilities="FARGATE")

The amount of memory (in MiB) used by the task. It can be expressed as an integer using MiB, for example `1024`, or as a string using GB, for example `1GB` or `1 GB`.

Example: `1024`, `"1GB"`, `"1 GB"`

`task-cpu` and `task-memory` must use one of the following values:

* 512 (0.5 GB), 1024 (1 GB), 2048 (2 GB) - Available cpu values: 256 (.25 vCPU)
* 1024 (1 GB), 2048 (2 GB), 3072 (3 GB), 4096 (4 GB) - Available cpu values: 512 (.5 vCPU)
* 2048 (2 GB), 3072 (3 GB), 4096 (4 GB), 5120 (5 GB), 6144 (6 GB), 7168 (7 GB), 8192 (8 GB) - Available cpu values: 1024 (1 vCPU)
* Between 4096 (4 GB) and 16384 (16 GB) in increments of 1024 (1 GB) - Available cpu values: 2048 (2 vCPU)
* Between 8192 (8 GB) and 30720 (30 GB) in increments of 1024 (1 GB) - Available cpu values: 4096 (4 vCPU)

### `network-mode` (optional)

The Docker networking mode to use for the containers in the task. The valid values are `none`, `bridge`, `awsvpc`, and `host`. The default Docker network mode is `bridge`.

Example: `"awsvpc"`

<<<<<<< HEAD
### `network-configuration` (optional)

The network configuration (The VPC subnets and security groups associated with a task) for the service. This parameter is required for task definitions that use the awsvpc network mode to receive their own elastic network interface, and it is not supported for other network modes.

Example: `"awsvpcConfiguration={subnets=[string,string],securityGroups=[string,string],assignPublicIp=string}"`
=======
### `fargate-memory` (optional)

If using `requires-compatibilities=FARGATE`, set the memory to be used.

### `fargate-cpu` (optional)

If using `requires-compatibilities=FARGATE`, set the CPU to be used.
>>>>>>> support-fargate

## AWS Roles

At a minimum this plugin requires the following AWS permissions to be granted to the agent running this step:

```yml
Policy:
  Statement:
  - Action:
    - ecr:DescribeImages
    - ecs:DescribeServices
    - ecs:RegisterTaskDefinition
    - ecs:UpdateService
    Effect: Allow
    Resource: '*'
```

This plugin will create the ECS Service if it does not already exist, which additionally requires the `ecs:CreateService` permission.

## Developing

To run the tests:

```bash
docker-compose run tests
```

## License

MIT (see [LICENSE](LICENSE))
