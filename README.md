# Topological Inventory Service OpenShift Specs and Templates

The `bin` directory contains scripts which will:
 - Create and start builds of the topological inventory service images (`bin/build`)
 - Deploy the topological inventory images into the current namespaces (`bin/deploy`)
 - Remove the topological inventory service (`bin/teardown`)

The build and deploy scripts require a single parameter which should be a file containing template parameters for use with `oc process`

## Template Parameters
These are the parameters that can be used when building and deploying the main application (not the collectors)

|  Name               |   Description                                | Default                          |
|---------------------|----------------------------------------------|----------------------------------|
| `APP_NAME`          | Name of the application                      | topological-inventory            |
| `PATH_PREFIX`       | API path to listen to                        |                                  |
| `IMAGE_NAMESPACE`   | Image namespace (used to build and deploy)   | buildfactory                     |
| `QUEUE_HOST`        | Queue host/service name                      |                                  |
| `DATABASE_PASSWORD` | PostgreSQL database password                 | generated from "[a-zA-Z0-9]{8}"  |
| `SECRET_KEY`        | Rails secret key                             | generated from "[a-f0-9]{128}"   |
| `ENCRYPTION_KEY`    | Encryption key for passwords in the database | generated from "[a-zA-Z0-9]{43}" |

---------------------------------------------------------------------------------------------------------

_**Notes on APP_NAME and PATH_PREFIX:**_

APP_NAME and PATH_PREFIX together will determine the base path that the API will listen on.
By default PATH_PREFIX is empty, if it is not specified, the application will listen on `/api/`
If PATH_PREFIX is specified, the application will listen on `/$PATH_PREFIX/$APP_NAME/`

## About Secrets

The deploy script is written so that it will not overwrite secrets.
This means that a secret will only be created or altered if it doesn't already exist or if that secret value is provided as a template parameter.

## Examples

### Build and deploy in a namespace called topological-inventory with an internal queue
Parameters file (`params-file`) contains the following:

```plain
QUEUE_HOST=topological-inventory-kafka
IMAGE_NAMESPACE=topological-inventory
```

1. Ensure you are working in the correct namespace

```bash
oc project topological-inventory
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
oc project topological-inventory
```

2. Deploy the things

```bash
bin/deploy params-file
```
