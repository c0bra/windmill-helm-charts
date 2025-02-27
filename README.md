<h1 align="center">
  <img src="https://media.licdn.com/dms/image/D4E0BAQFh9M8zrNFjQw/company-logo_200_200/0/1665352677142?e=2147483647&v=beta&t=iI4YPPusisIuK3I-VjYnt7WVuIA4jsSQpglIwfr9X2U" alt="windmill" width="100">
</h1>

<h4 align="center">Turn scripts into workflows and UIs in minutes</h4>

<p align="center">
<img src="https://github.com/windmill-labs/windmill-helm-charts/actions/workflows/helm_test.yml/badge.svg" alt="drawing"/>
<img src="https://github.com/windmill-labs/windmill-helm-charts/actions/workflows/release.yml/badge.svg" alt="drawing"/>
<img src="https://img.shields.io/github/v/release/windmill-labs/windmill-helm-charts" alt="drawing"/>
<img src="https://img.shields.io/github/downloads/windmill-labs/windmill-helm-charts/total.svg" alt="drawing"/>
</p>

<p align="center">
  <a href="#install">Install</a> •
  <a href="#core-values">Core Values</a> •
  <a href="#full-values">Full Values</a> •
  <a href="#local-s3">Local S3</a> •
  <a href="#caveats">Caveats</a> •  
  <a href="#k8s-tips">Kubernetes Tips</a>
</p>

<hr>

## Install

> Have Helm 3 [installed](https://helm.sh/docs/intro/install).

```sh
helm repo add windmill https://windmill-labs.github.io/windmill-helm-charts/
helm install mywindmill windmill/windmill -n windmill --create-namespace --values values.yaml
```

To update versions:

```
helm repo update windmill
helm upgrade mywindmill windmill/windmill -n windmill --values values.yaml
```

You do not need to provide a values.yaml to be able to test it on minikube.
Follow the steps below.

### When using a non super-user role for postgresql in databaseUrl

You will need to setup some required roles which would otherwise be done
automatically when using a super-user role for the databaseUrl.

Follow those
[instructions](https://docs.windmill.dev/docs/advanced/self_host#run-windmill-without-using-a-postgres-superuser)

### Test it on minikube

To make it work on a local minkube to test. Get the ip address of the ingress:

```
▶ kubectl get ingress -n windmill
NAME       CLASS    HOSTS                        ADDRESS        PORTS   AGE
windmill   <none>   windmill,windmill,windmill   192.168.49.2   80      13m
```

If not ip address is displayed, enable the ingress addon:

```
minikube addons enable ingress
```

Then modify /etc/hosts to match the `baseDomain`, by default 'windmill'.

E.g:

```
192.168.49.2   windmill
```

Then open your browser at <http://windmill>

## Core Values

```yaml
# windmill root values block
windmill:
  # domain as shown in browser, this is used together with `baseProtocol` as part of the BASE_URL environment variable in app and worker container and in the ingress resource, if enabled
  baseDomain: windmill
  baseProtocol: http
  # postgres URI, pods will crashloop if database is unreachable, sets DATABASE_URL environment variable in app and worker container
  databaseUrl: postgres://postgres:windmill@windmill-postgresql/windmill?sslmode=disable
  # replica for the application app
  appReplicas: 2
  # replicas for the workers, jobs are executed on the workers
  workerReplicas: 2
  # replicas for the lsp containers used by the app
  lspReplicas: 2
  # add additional worker groups
  workerGroups:
  # workers configuration
  - name: "gpu"
    ...
  # Use those to override the tag or image used for the app and worker containers. Windmill uses the same image for both.
  # By default, if enterprise is enable, the image is set to ghcr.io/windmill-labs/windmill-ee, otherwise the image is set to ghcr.io/windmill-labs/windmill
  #tag: "mytag"
  #image: "ghcr.io/windmill-labs/windmill"
  ...

# enable postgres (bitnami) on kubernetes
postgresql:
  enabled: true

# enable minio (bitnami) on kubernetes
minio:
  enabled: true

# Configure Ingress
ingress:
  className: ""
  ...

# enable enterprise features
enterprise:
  # -- enable windmill enterprise, requires license key.
  enabled: false
  # -- s3 bucket to use for dependency cache. Sets S3_CACHE_BUCKET environment variable in worker container
  s3CacheBucket: mybucketname
  # -- windmill provided Enterprise license key. Sets LICENSE_KEY environment variable in app and worker container.
  licenseKey: 123456F
  # -- use nsjail for sandboxing
  nsjail: false
```

## Full Values

| Key                                                             | Type   | Default                                                                                                                                                    | Description                                                                                                                                                                                                      |
| --------------------------------------------------------------- | ------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| enterprise.enabled                                              | bool   | `false`                                                                                                                                                    | enable Windmill Enterprise , requires license key.                                                                                                                                                               |
| enterprise.enabledS3DistributedCache                            | bool   | `false`                                                                                                                                                    |                                                                                                                                                                                                                  |
| enterprise.licenseKey                                           | string | `"123456F"`                                                                                                                                                | Windmill provided Enterprise license key. Sets LICENSE_KEY environment variable in app and worker container.                                                                                                     |
| enterprise.nsjail                                               | bool   | `false`                                                                                                                                                    | use nsjail for sandboxing                                                                                                                                                                                        |
| enterprise.s3CacheBucket                                        | string | `"mybucketname"`                                                                                                                                           | S3 bucket to use for dependency cache. Sets S3_CACHE_BUCKET environment variable in worker container                                                                                                             |
| enterprise.samlMetadata                                         | string | `""`                                                                                                                                                       | SAML Metadata URL to enable SAML SSO                                                                                                                                                                             |
| enterprise.scimToken                                            | string | `""`                                                                                                                                                       |                                                                                                                                                                                                                  |
| ingress.annotations                                             | object | `{}`                                                                                                                                                       |                                                                                                                                                                                                                  |
| ingress.className                                               | string | `""`                                                                                                                                                       |                                                                                                                                                                                                                  |
| ingress.enabled                                                 | bool   | `true`                                                                                                                                                     | enable/disable included ingress resource                                                                                                                                                                         |
| ingress.tls                                                     | list   | `[]`                                                                                                                                                       | TLS config for the ingress resource. Useful when using cert-manager and nginx-ingress                                                                                                                            |
| minio.auth.rootPassword                                         | string | `"windmill"`                                                                                                                                               |                                                                                                                                                                                                                  |
| minio.auth.rootUser                                             | string | `"windmill"`                                                                                                                                               |                                                                                                                                                                                                                  |
| minio.enabled                                                   | bool   | `false`                                                                                                                                                    | enabled included Minio operator for s3 resource demo purposes                                                                                                                                                    |
| minio.fullnameOverride                                          | string | `"windmill-minio"`                                                                                                                                         |                                                                                                                                                                                                                  |
| minio.mode                                                      | string | `"standalone"`                                                                                                                                             |                                                                                                                                                                                                                  |
| minio.primary.enabled                                           | bool   | `true`                                                                                                                                                     |                                                                                                                                                                                                                  |
| postgresql.auth.database                                        | string | `"windmill"`                                                                                                                                               |                                                                                                                                                                                                                  |
| postgresql.auth.postgresPassword                                | string | `"windmill"`                                                                                                                                               |                                                                                                                                                                                                                  |
| postgresql.enabled                                              | bool   | `true`                                                                                                                                                     | enabled included Postgres container for demo purposes only using bitnami                                                                                                                                         |
| postgresql.fullnameOverride                                     | string | `"windmill-postgresql"`                                                                                                                                    |                                                                                                                                                                                                                  |
| postgresql.primary.persistence.enabled                          | bool   | `true`                                                                                                                                                     |                                                                                                                                                                                                                  |
| serviceAccount.annotations                                      | object | `{}`                                                                                                                                                       |                                                                                                                                                                                                                  |
| serviceAccount.create                                           | bool   | `true`                                                                                                                                                     |                                                                                                                                                                                                                  |
| serviceAccount.name                                             | string | `""`                                                                                                                                                       |                                                                                                                                                                                                                  |
| windmill.app.affinity                                           | object | `{}`                                                                                                                                                       | Affinity rules to apply to the pods                                                                                                                                                                              |
| windmill.app.annotations                                        | object | `{}`                                                                                                                                                       | Annotations to apply to the pods                                                                                                                                                                                 |
| windmill.app.autoscaling.enabled                                | bool   | `false`                                                                                                                                                    | enable or disable autoscaling                                                                                                                                                                                    |
| windmill.app.autoscaling.maxReplicas                            | int    | `10`                                                                                                                                                       | maximum autoscaler replicas                                                                                                                                                                                      |
| windmill.app.autoscaling.targetCPUUtilizationPercentage         | int    | `80`                                                                                                                                                       | target CPU utilization                                                                                                                                                                                           |
| windmill.app.extraEnv                                           | list   | `[]`                                                                                                                                                       | Extra environment variables to apply to the pods                                                                                                                                                                 |
| windmill.app.nodeSelector                                       | object | `{}`                                                                                                                                                       | Node selector to use for scheduling the pods                                                                                                                                                                     |
| windmill.app.resources                                          | object | `{}`                                                                                                                                                       | Resource limits and requests for the pods                                                                                                                                                                        |
| windmill.app.tolerations                                        | list   | `[]`                                                                                                                                                       | Tolerations to apply to the pods                                                                                                                                                                                 |
| windmill.appReplicas                                            | int    | `2`                                                                                                                                                        | replica for the application app                                                                                                                                                                                  |
| windmill.baseDomain                                             | string | `"windmill"`                                                                                                                                               | domain as shown in browser, this variable and `baseProtocol` are used as part of the BASE_URL environment variable in app and worker container and in the ingress resource, if enabled                           |
| windmill.baseProtocol                                           | string | `"http"`                                                                                                                                                   | protocol as shown in browser, change to https etc based on your endpoint/ingress configuration, this variable and `baseDomain` are used as part of the BASE_URL environment variable in app and worker container |
| windmill.cookieDomain                                           | string | `""`                                                                                                                                                       | domain to use for the cookies. Use it if windmill is hosted on a subdomain and you need to share the cookies with the hub for instance                                                                           |
| windmill.createWorkspaceRequireSuperadmin                       | bool   | `false`                                                                                                                                                    | is any user allowed to create workspaces                                                                                                                                                                         |
| windmill.databaseUrl                                            | string | `"postgres://postgres:windmill@windmill-postgresql/windmill?sslmode=disable"`                                                                              | Postgres URI, pods will crashloop if database is unreachable, sets DATABASE_URL environment variable in app and worker container                                                                                 |
| windmill.databaseUrlSecretName                                  | string | `""`                                                                                                                                                       | name of the secret storing the database URI, take precedence over databaseUrl. The key of the url is 'url'                                                                                                       |
| windmill.exposeHostDocker                                       | bool   | `false`                                                                                                                                                    | mount the docker socket inside the container to be able to run docker command as docker client to the host docker daemon                                                                                         |
| windmill.globalErrorHandlerPath                                 | string | `""`                                                                                                                                                       | if set, the path to a script in the admins workspace that will be triggered upon any jobs failure                                                                                                                |
| windmill.image                                                  | string | `""`                                                                                                                                                       | windmill image tag, will use the Acorresponding ee or ce image from ghcr if not defined. Do not include tag in the image name.                                                                                   |
| windmill.instanceEventsWebhook                                  | string | `""`                                                                                                                                                       | send instance events to a webhook. Can be hooked back to windmill                                                                                                                                                |
| windmill.lsp.affinity                                           | object | `{}`                                                                                                                                                       | Affinity rules to apply to the pods                                                                                                                                                                              |
| windmill.lsp.annotations                                        | object | `{}`                                                                                                                                                       | Annotations to apply to the pods                                                                                                                                                                                 |
| windmill.lsp.autoscaling.enabled                                | bool   | `false`                                                                                                                                                    | enable or disable autoscaling                                                                                                                                                                                    |
| windmill.lsp.autoscaling.maxReplicas                            | int    | `10`                                                                                                                                                       | maximum autoscaler replicas                                                                                                                                                                                      |
| windmill.lsp.autoscaling.targetCPUUtilizationPercentage         | int    | `80`                                                                                                                                                       | target CPU utilization                                                                                                                                                                                           |
| windmill.lsp.extraEnv                                           | list   | `[]`                                                                                                                                                       | Extra environment variables to apply to the pods                                                                                                                                                                 |
| windmill.lsp.nodeSelector                                       | object | `{}`                                                                                                                                                       | Node selector to use for scheduling the pods                                                                                                                                                                     |
| windmill.lsp.resources                                          | object | `{}`                                                                                                                                                       | Resource limits and requests for the pods                                                                                                                                                                        |
| windmill.lsp.tag                                                | string | `"latest"`                                                                                                                                                 |                                                                                                                                                                                                                  |
| windmill.lsp.tolerations                                        | list   | `[]`                                                                                                                                                       | Tolerations to apply to the pods                                                                                                                                                                                 |
| windmill.lspReplicas                                            | int    | `2`                                                                                                                                                        | replicas for the lsp containers used by the app                                                                                                                                                                  |
| windmill.multiplayer.affinity                                   | object | `{}`                                                                                                                                                       | Affinity rules to apply to the pods                                                                                                                                                                              |
| windmill.multiplayer.annotations                                | object | `{}`                                                                                                                                                       | Annotations to apply to the pods                                                                                                                                                                                 |
| windmill.multiplayer.autoscaling.enabled                        | bool   | `false`                                                                                                                                                    | enable or disable autoscaling                                                                                                                                                                                    |
| windmill.multiplayer.autoscaling.maxReplicas                    | int    | `10`                                                                                                                                                       | maximum autoscaler replicas                                                                                                                                                                                      |
| windmill.multiplayer.autoscaling.targetCPUUtilizationPercentage | int    | `80`                                                                                                                                                       | target CPU utilization                                                                                                                                                                                           |
| windmill.multiplayer.extraEnv                                   | list   | `[]`                                                                                                                                                       | Extra environment variables to apply to the pods                                                                                                                                                                 |
| windmill.multiplayer.nodeSelector                               | object | `{}`                                                                                                                                                       | Node selector to use for scheduling the pods                                                                                                                                                                     |
| windmill.multiplayer.resources                                  | object | `{}`                                                                                                                                                       | Resource limits and requests for the pods                                                                                                                                                                        |
| windmill.multiplayer.tag                                        | string | `"latest"`                                                                                                                                                 |                                                                                                                                                                                                                  |
| windmill.multiplayer.tolerations                                | list   | `[]`                                                                                                                                                       | Tolerations to apply to the pods                                                                                                                                                                                 |
| windmill.multiplayerReplicas                                    | int    | `2`                                                                                                                                                        | replicas for the lsp containers used by the app                                                                                                                                                                  |
| windmill.npmConfigRegistry                                      | string | `""`                                                                                                                                                       | pass the npm for private registries                                                                                                                                                                              |
| windmill.numWorkers                                             | int    | `1`                                                                                                                                                        | workers per worker container, default and recommended is 1 to isolate one process per container, sets NUM_WORKER environment variable for worker container. app container has 0 NUM_WORKERS by default           |
| windmill.oauthConfig                                            | string | `"{}\n"`                                                                                                                                                   | raw oauth config. See https://docs.windmill.dev/docs/misc/setup_oauth                                                                                                                                            |
| windmill.oauthSecretName                                        | string | `""`                                                                                                                                                       | name of the secret storing the oauthConfig. See https://docs.windmill.dev/docs/misc/setup_oauth                                                                                                                  |
| windmill.pipExtraIndexUrl                                       | string | `""`                                                                                                                                                       | pass the extra index url to pip for private registries                                                                                                                                                           |
| windmill.pipIndexUrl                                            | string | `""`                                                                                                                                                       | pass the index url to pip for private registries                                                                                                                                                                 |
| windmill.pipTrustedHost                                         | string | `""`                                                                                                                                                       | pass the trusted host to pip for private registries                                                                                                                                                              |
| windmill.rustLog                                                | string | `"info"`                                                                                                                                                   | rust log level, set to debug for more information etc, sets RUST_LOG environment variable in app and worker container                                                                                            |
| windmill.smtp                                                   | object | `{"enabled":false,"from":"noreply@windmill.dev","host":"smtp.gmail.com","password":"bar","port":587,"tls_implicit":false,"username":"ruben@windmill.dev"}` | set smtp configs for windmill to be able to send emails when adding users to instance and workspaces                                                                                                             |
| windmill.tag                                                    | string | `""`                                                                                                                                                       | windmill app image tag, will use the App version if not defined                                                                                                                                                  |
| windmill.workerGroups[0].affinity                               | object | `{}`                                                                                                                                                       | Affinity rules to apply to the pods                                                                                                                                                                              |
| windmill.workerGroups[0].annotations                            | object | `{}`                                                                                                                                                       | Annotations to apply to the pods                                                                                                                                                                                 |
| windmill.workerGroups[0].extraEnv                               | list   | `[]`                                                                                                                                                       | Extra environment variables to apply to the pods                                                                                                                                                                 |
| windmill.workerGroups[0].name                                   | string | `"gpu"`                                                                                                                                                    |                                                                                                                                                                                                                  |
| windmill.workerGroups[0].nodeSelector                           | object | `{}`                                                                                                                                                       | Node selector to use for scheduling the pods                                                                                                                                                                     |
| windmill.workerGroups[0].replicas                               | int    | `0`                                                                                                                                                        |                                                                                                                                                                                                                  |
| windmill.workerGroups[0].resources                              | object | `{}`                                                                                                                                                       | Resource limits and requests for the pods                                                                                                                                                                        |
| windmill.workerGroups[0].tolerations                            | list   | `[]`                                                                                                                                                       | Tolerations to apply to the pods                                                                                                                                                                                 |
| windmill.workerReplicas                                         | int    | `2`                                                                                                                                                        | replicas for the workers, jobs are executed on the workers                                                                                                                                                       |
| windmill.workers.affinity                                       | object | `{}`                                                                                                                                                       | Affinity rules to apply to the pods                                                                                                                                                                              |
| windmill.workers.annotations                                    | object | `{}`                                                                                                                                                       | Annotations to apply to the pods                                                                                                                                                                                 |
| windmill.workers.autoscaling.enabled                            | bool   | `false`                                                                                                                                                    | will not benefit from the global cache and the performances will be poor for newly spawned pods                                                                                                                  |
| windmill.workers.autoscaling.maxReplicas                        | int    | `10`                                                                                                                                                       | maximum autoscaler replicas                                                                                                                                                                                      |
| windmill.workers.autoscaling.targetCPUUtilizationPercentage     | int    | `80`                                                                                                                                                       | target CPU utilization                                                                                                                                                                                           |
| windmill.workers.extraEnv                                       | list   | `[]`                                                                                                                                                       | Extra environment variables to apply to the pods                                                                                                                                                                 |
| windmill.workers.nodeSelector                                   | object | `{}`                                                                                                                                                       | Node selector to use for scheduling the pods                                                                                                                                                                     |
| windmill.workers.resources                                      | object | `{}`                                                                                                                                                       | Resource limits and requests for the pods                                                                                                                                                                        |
| windmill.workers.tolerations                                    | list   | `[]`                                                                                                                                                       | Tolerations to apply to the pods                                                                                                                                                                                 |

## Local S3

The chart includes a Minio S3 distribution to demonstrate the usage of S3 as a
resource in a vendor-agnostic environment like Kubernetes. The local Minio S3
service will be available to the Windmill workers through its Kubernetes
service, which is set to "windmill-minio" by default. In the Resources page, you
should create an S3 API Connection Object, and import it as a connection object
to reduce code duplication between scripts. For the sake of this example, this
stage is skipped. Below is an example of how to authenticate and use the
provided local S3 distribution in a Python script running in Windmill:

```python
from minio import Minio

def main():
    # Create a client with the MinIO server, its access key
    # and secret key.
    client = Minio(
        "windmill-minio", # Local Kubernetes Service
        access_key="windmill",
        secret_key="windmill",
    )

    # Make 'demo' bucket if not exist.
    found = client.bucket_exists("demo")
    if not found:
        client.make_bucket("demo")
    else:
        print("Bucket 'demo' already exists")

    with open('readme.txt', 'w') as f:
        f.write('Create a new text file!')

    client.fput_object(
        "demo", "readme.txt", "readme.txt",
    )

    print(
        "'readme.txt' is successfully uploaded as "
        "object 'readme.txt' to bucket 'demo'."
    )
```

## Enterprise features

To use the enterprise version with the <license key> provided upon subscription,
add the following to the values.yaml file:

```
enterprise:
  enabled: true
  licenseKey: <license key>
```

### S3 Cache

Enterprise users can use S3 storage for dependency caching for performance.
Cache is two way synced at regular intervals (10 minutes). To use it, the worker
deployment requires access to an S3 bucket. There are several ways to do this:

1. On AWS (and EKS) , you can use a service account with IAM roles attached. See
   [AWS docs](https://docs.aws.amazon.com/eks/latest/userguide/associate-service-account-role.html) -
   once you have a policy , you can create an account via eksctl for instance

   ```sh
   eksctl create iamserviceaccount --name serviceaccountname --namespace production --cluster windmill-cluster --role-name "iamrolename" \ --attach-policy-arn arn:aws:iam::12312315:policy/bucketpolicy --approve
   ```

2. Mount/attach a credentials file in `/root/.aws/credentials` of the worker
   deployment
3. Add environment variables for the `AWS_ACCESS_KEY_ID` and
   `AWS_SECRET_ACCESS_KEY`, via kube secrets.

The sync relies on rclone and uses its methods of authentication to s3 per
[Rclone documentation](https://rclone.org/s3/#authentication)

Then the values settings become:

```
enterprise:
  enabled: true
  licenseKey: <license key>
  enabledS3DistributedCache: true
  s3CacheBucket: mybucketname
```

## Caveats

- Postgres is included for demo purposes, it is a stateful set with a small
  volume claim applied. If you want to host postgres in k8s, there are better
  ways, or offload it outside your k8s cluster. Postgres can be disabled
  entirely in the values.yaml file.
- The postgres user/pass is currently not a secret/encrypted

## Kubernetes Hosting Tips

The helm chart does have an ingress configuration included. It's enabled by
default. The ingress uses the `windmill.baseDomain` variable for its hostname
configuration. Here are two example configurations for an AWS ALB and
nginx-ingress/cert-manager:

AWS ALB:

```yaml
windmill:
  baseDomain: "windmill.example.com"
  ...

ingress:
  className: "alb"
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/tags: Environment=dev,Team=test
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/target-group-attributes: stickiness.enabled=true,stickiness.lb_cookie.duration_seconds=604800,stickiness.type=lb_cookie
    alb.ingress.kubernetes.io/load-balancer-attributes: idle_timeout.timeout_seconds=600
    alb.ingress.kubernetes.io/group.name: windmill
    alb.ingress.kubernetes.io/group.order: '10'
    alb.ingress.kubernetes.io/certificate-arn: my-certificatearn
...
```

nginx ingress + cert-manager:

```yaml
windmill:
  baseDomain: "windmill.example.com"
  ...

ingress:
  className: "nginx"
  tls:
    - hosts:
        - "windmill.example.com"
      secretName: windmill-tls-cert
  annotations:
    cert-manager.io/issuer: "letsencrypt-prod"
    nginx.ingress.kubernetes.io/affinity: "cookie"
    nginx.ingress.kubernetes.io/affinity-mode: "persistent"
    nginx.ingress.kubernetes.io/session-cookie-name: "route"
...
```

There are many ways to expose an app and it will depend on the requirements of
your environment. If you don't want to use the included ingress and roll your
own, you can just disable it. Overall, you want the following endpoints
accessible included in the chart:

windmill app on port 8000 lsp application on port 3001 metrics endpoints on port
8001 for the app/app and workers If you are using Prometheus, you can scrape the
windmill-app-metrics service on port 8001 at /metrics endpoint to gather stats
about the Windmill application.

A ServiceMonitor is included in the chart for Prometheus Operator users.
