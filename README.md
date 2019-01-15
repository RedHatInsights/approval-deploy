# Approval Service OpenShift Specs and Templates

The `bin` directory contains scripts which will:
 - Create and start builds of the approval service images (`bin/build`)
 - Deploy the approval images into the current namespaces (`bin/deploy`)
 - Remove the approval service (`bin/teardown`)

The build and deploy scripts require a single parameter which should be a file containing template parameters for use with `oc process`

## Template Parameters
These are the parameters that can be used when building and deploying the application

| Name                           | Description                                             | Default                          |
|--------------------------------|---------------------------------------------------------|----------------------------------|
| `APP_NAME`                     | Name of the application                                 | approval                         |
| `PATH_PREFIX`                  | API path to listen to                                   |                                  |
| `IMAGE_NAMESPACE`              | Image namespace (used to build and deploy)              | buildfactory                     |
| `QUEUE_HOST`                   | Queue host/service name                                 |                                  |
| `DATABASE_PASSWORD`            | PostgreSQL database password                            | generated from "[a-zA-Z0-9]{8}"  |
| `SECRET_KEY`                   | Rails secret key                                        | generated from "[a-f0-9]{128}"   |

---------------------------------------------------------------------------------------------------------

_**Notes on APP_NAME and PATH_PREFIX:**_

APP_NAME and PATH_PREFIX together will determine the base path that the API will listen on.
By default PATH_PREFIX is empty, if it is not specified, the application will listen on `/api/`
If PATH_PREFIX is specified, the application will listen on `/$PATH_PREFIX/$APP_NAME/`

## About Secrets

The deploy script is written so that it will not overwrite secrets.
This means that a secret will only be created or altered if it doesn't already exist or if that secret value is provided as a template parameter.

## Examples

These examples assume the approval service will be running in the same namespace and the topological inventory service will be running in a namespace called "topological-inventory" within the same cluster.

### Build and deploy in a namespace called approval with an internal queue
Parameters file (`params-file`) contains the following:

```plain
IMAGE_NAMESPACE=approval
QUEUE_HOST=approval-kafka
```

1. Ensure you are working in the correct namespace

```bash
oc project approval
```

2. Create and run builds

```bash
bin/build params-file
```

3. Deploy the queue

```bash
oc apply -f queue/deployment_config.yaml
oc apply -f queue/service.yaml
```

4. Deploy all the other things

```bash
bin/deploy params-file
```

### Deploy from buildfactory with an external queue service and a path prefix
Parameters file (`params-file`) contains the following:

```plain
QUEUE_HOST=platform-mq-ci-kafka-bootstrap.platform-mq-ci.svc
PATH_PREFIX=r/insights/platform
```

1. Ensure you are working in the correct namespace

```bash
oc project approval
```

2. Deploy the things

```bash
bin/deploy params-file
```
