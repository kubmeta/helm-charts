# How to

### Add repository

```
helm repo add kubmeta https://charts.kubmeta.com/
```

### Install chart
```
helm install my-app kubmeta/kubmeta-lib


```

```
dependencies:
  - name: kubmeta-lib
    version: 3.1.10
    repository: https://charts.kubmeta.com/
```

```bash
$ helm dependency update
$ helm install my-app .
```

## Introduction

This chart provides a kubmeta-lib template helpers which can be used to develop new charts using [Helm](https://helm.sh) package manager.

## Prerequisites

- Kubernetes 1.18+
- Helm 3.7.1

## \_helpers.tpl

| Helper identifier         | Description                                                                                   | Expected Input              | Default Value             |
| ------------------------- | --------------------------------------------------------------------------------------------- | --------------------------- | ------------------------- |
| `kubmeta-lib.name`               | The name of the chart.                                                                        | .Values.name                | .Chart.Name               |
| `kubmeta-lib.fullname`           | Fully qualified app name. If release name contains chart name it will be used as a full name. | .Values.fullnameOverride    | .Release.Name             |
| `kubmeta-lib.version`            | The version of the chart.                                                                     | .Values.version             | .Chart.Version            |
| `kubmeta-lib.appVersion`         | The appVersion of the chart.                                                                  | .Values.appVersion          | .Chart.AppVersion         |
| `kubmeta-lib.chart`              | It's made of kubmeta-lib.name and kubmeta-lib.version.                                                      | ---                         | ---                       |
| `kubmeta-lib.labels`             | Common labels.                                                                                | ---                         | ---                       |
| `kubmeta-lib.selectorLabels`     | Selector labels.                                                                              | ---                         | ---                       |
| `kubmeta-lib.serviceAccountName` | The name of the service account to use.                                                       | .Values.serviceAccount.name | kubmeta-lib.fullname / "default" |

## Default Values

```
replicaCount: 1
image:
  repository: nginx
  pullPolicy: IfNotPresent
  tag: 1.21
imagePullSecrets: []
nameOverride: ""
fullnameOverride: ""
serviceAccount:
  create: true
  annotations: {}
  name: ""
podAnnotations: {}
podSecurityContext: {}
securityContext: {}
service:
  type: NodePort
  port: 80
ingress:
  enabled: false
  annotations: {}
  hosts:
    - host: host.name.com
      paths:
      - path: /*
        backend:
          serviceName: ssl-redirect
          servicePort: use-annotation
      - backend:
          serviceName: kubmeta-lib
          servicePort: 80
        path: /*
  tls: []
resources: {}
autoscaling:
  enabled: false
  minReplicas: 1
  maxReplicas: 100
  targetCPUUtilizationPercentage: 80
nodeSelector: {}
tolerations: []
affinity: {}
config: {}
containerPort: 80
livenessProbe: {}
readinessProbe: {}
deployment: {}
env: dev
product: application
secrets: {}
```

## Problems

If you want your chart to have the name `kubmeta-lib` you have to specify it in Chart.yaml file by passing it to `alias`. After everything has to be defined under the alias name(in this case under `kubmeta-lib`) in your values.yaml file. Otherwise, it will get the name `kubmeta-lib`. It's a problem that has no dynamic solution in helm. The same problem is with the version and appVersion. Your parent chart will receive `kubmeta-lib`'s version if you do not change them by these 2 variables: `version`, `appVersion`. None of these is a mandatory value, so without them, you will have no problem in the process of running the chart.
Here are 2 examples about this:

### Example 1

**Alias is not specified, name, version and appVersion are not overridden by parent chart**
In this case, labels will be like this:\*\*

- `helm.sh/chart: kubmeta-lib-3.1.10 `
- `app.kubernetes.io/name: kubmeta-lib`
- `app.kubernetes.io/version: 3.1.10 `

Chart.yaml

```
apiVersion: v2
name: my-application
description: My application service description
type: application
version: 3.1.10
appVersion: "3.1.10"

dependencies:
  - name: kubmeta-lib
    version: 3.1.10
    repository: https://kubmeta.github.io/helm-charts
```

values.yaml

These are mandatory values you should provide.

```
kubmeta-lib:
  image:
    repository: docker-image
    tag: 1.2.3
```

### Example 2

**Alias is specified/or name is overriden, also version and appVersion are overridden by parent chart**
Now if you prefer to change your chart's name and version, you can do it by the alias variable(passing a name to it in Chart.yaml) or you can override the `name` in values.yaml file. The result will be the same. About `version` and `appVersion`, they have to be changed in values.yaml.
The same labels, given above, now will be:

- `helm.sh/chart: kubmeta-lib-3.1.10`
- `app.kubernetes.io/name: kubmeta-lib`
- `app.kubernetes.io/version: 3.1.10`

**This is with the alias name.**

Chart.yaml

```
apiVersion: v2
name: my-application
description: My application service description
type: application
version: 3.1.10
appVersion: "3.1.10"

dependencies:
  - name: kubmeta-lib
    version: 3.1.10
    repository: https://kubmeta.github.io/helm-charts
    alias: kubmeta-lib
```

values.yaml

```
kubmeta-lib:
  version: 3.1.10
  appVersion: 3.1.10
  image:
    repository: docker-image
    tag: 1.2.3
```

**This is by overridding the `name`.**

Chart.yaml

```
apiVersion: v2
name: my-application
description: My application service description
type: application
version: 3.1.10
appVersion: "3.1.10"

dependencies:
  - name: kubmeta-lib
    version: 3.1.10
    repository: https://kubmeta.github.io/helm-charts
```

values.yaml

```
kubmeta-lib:
  name: kubmeta-lib
  version: 3.1.10
  appVersion: 3.1.10
  image:
    repository: docker-image
    tag: 1.2.3
```

### External secrets

This will produce external secrets which will ask secrets from store `production`.
Values in Secrets Manager should be put in `my-product/production/my-app` matching secret1, secret2, secret2.
You have to previously create a Secret Store. Check our `external-secret-store` module in Terraform Registry.

```
kubmeta-lib:
  ...
  product: my-product
  env: production
  secrets:
    - secret1
    - secret2
    - secret3
```

### Volumes

**The PVC has a specific name**

```
kubmeta-lib:
  ...
  deployment:
    volumes:
    - name: test-volume
      persistentVolumeClaim:
        claimName: volume-1
```

**The PVC has the name of the release**

```
kubmeta-lib:
  ...
  deployment:
    volumes:
    - name: test-volume
      persistentVolumeClaim:
        dynamicName: true
```

### PVC

By defining `storage` block in values.yaml file you can create a PVC resource.

```
kubmeta-lib:
  ...
  storage:
    className: aws-efs
    accessModes:
      - ReadWriteOnce
    requestedSize: 4Gi
```

### Health Checks

```
livenessProbe: {}
  failureThreshold: 3
  httpGet:
    path: /
    port: 80
    scheme: HTTP
  initialDelaySeconds: 60
  periodSeconds: 5
  successThreshold: 1
  timeoutSeconds: 1

readinessProbe: {}
  httpGet:
    path: /
    port: http
  initialDelaySeconds: 60
  periodSeconds: 5
```
