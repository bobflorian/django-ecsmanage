# django-ecsmanage

A Django app that provides a management command allowing you to run any
other management command on an AWS Elastic Container Service
(ECS) cluster.

With `django-ecsmanage`, you can easily run migrations and other one-off tasks
on a remote cluster from the command line:

```
$ django-admin ecsmanage migrate
```

## Table of Contents

- [Installation](#installation)
- [Configuration](#configuration)
  - [Environments](#environments)
  - [AWS Resources](#aws-resources)
- [Developing](#developing)

## Installation

Install from PyPi using pip:

```
$ pip install django-ecsmanage
```

Update `INSTALLED_APPS` in your Django settings to install the app:

```python
INSTALLED_APPS = (
    ...
    'ecsmanage',
)
```

## Configuration 

Settings for the management command are kept in a single configuration
dictionary in your Django settings named `ECSMANAGE_ENVIRONMENTS`. Each entry in
`ECSMANAGE_ENVIRONMENTS` should be a key-value pair corresponding to a
named environment (like `default` or `production`) and a set of AWS resources
associated with that environment. For example:

```python
ECSMANAGE_ENVIRONMENTS = {
    'default': {
        'TASK_DEFINITION_NAME': 'StagingAppCLI',
        'CLUSTER_NAME': 'ecsStagingCluster',
        'LAUNCH_TYPE': 'FARGATE',
        'SECURITY_GROUP_TAGS': {
            'Name': 'sgAppEcsService',
            'Environment': 'Staging',
            'Project': 'ProjectName'
        },
        'SUBNET_TAGS': {
            'Name': 'PrivateSubnet',
            'Environment': 'Staging',
            'Project': 'ProjectName'
        },
        'AWS_REGION': 'us-east-1',
    },
}
```

This configuration defines a single environment, named `default`, with
associated AWS ECS resources.

### Environments

The key name for an environment can be any string. You can use this name
with the `--env` flag when running the command to run a command on a
different environment. Take this `ECSMANAGE_ENVIRONMENTS` configuration
as an example:

```python
ECSMANAGE_ENVIRONMENTS = {
    'default': {
        'TASK_DEFINITION_NAME': 'StagingAppCLI',
        'CLUSTER_NAME': 'ecsStagingCluster',
        'SECURITY_GROUP_TAGS': {
            'Name': 'sgStagingAppEcsService',
        },
        'SUBNET_TAGS': {
            'Name': 'StagingPrivateSubnet',
        },
    },
    'production': {
        'TASK_DEFINITION_NAME': 'ProductionAppCLI',
        'CLUSTER_NAME': 'ecsProductionCluster',
        'SECURITY_GROUP_TAGS': {
            'Name': 'sgProductionAppEcsService',
        },
        'SUBNET_TAGS': {
            'Name': 'ProductionPrivateSubnet',
        },
    },
}
```

This configuration defines two environments, `default` and `production`.
Using the above settings, you could run production migrations with the
following command:

```
$ django-admin ecsmanage --env production migrate
```

If the `--env` argument is not present, the command will default to the
environment named `default`. 

### AWS Resources

The following environment configuration keys help the management command locate
the appropriate AWS resources for your cluster:

| Key                    | Description                                                                                          | Default     |
| ---------------------- | ---------------------------------------------------------------------------------------------------- | ----------- |
| `TASK_DEFINITION_NAME` | The name of your ECS task definition. The command will automatically retrieve the latest definition. |             |
| `CLUSTER_NAME`         | The name of your ECS cluster.                                                                        |             |
| `SECURITY_GROUP_TAGS`  | A dictionary of tags to use to identify a security group for your task.                              |             |
| `SUBNET_TAGS`          | A dictionary of tags to use to identify a subnet for your task.                                      |             |
| `LAUNCH_TYPE`          | The ECS launch type for your task.                                                                   | `FARGATE`   |
| `AWS_REGION`           | The AWS region to run your task.                                                                     | `us-east-1` |

## Developing

Local development is managed with Python virtual environments. Make sure that
you have [Python 3.6+ and pip installed](https://www.python.org/downloads/)
before starting.

Install the development package in a virtual environment:

```
$ ./scripts/update
```

Run the tests:

```
$ ./scripts/test
```
