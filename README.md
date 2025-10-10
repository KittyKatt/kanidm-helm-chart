# Kanidm Helm Chart (non-official)

![Version: 0.1.0](https://img.shields.io/badge/Version-0.1.0-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square) ![AppVersion: 1.4.5](https://img.shields.io/badge/AppVersion-1.4.5-informational?style=flat-square)

## Introduction

This is a non-official Helm chart for the [Kanidm identity management platform](https://github.com/kanidm/kanidm). I use Kanidm exlusively for all of my identity needs with anything that speaks OIDC (and some that I bully into speaking OIDC). This was made mostly to suit my needs and use case, but I tried to work in some other use cases I could think of. If something about this chart doesn't meet the needs of your specific environment/deployment, feel free to open a PR and maybe we can adapt the chart to allow it. Enjoy.

## Prerequisites

* Tested against Kubernetes v1.30.1
* Tested with Helm v3.16.3
* If you want to run unit tests, you'll need to install the Helm unittests plugin. See the [Testing](#testing) section for more information.

## Installation

### Add Helm Repository

TBD, you'll have to install by cloning the chart for now

### Configuration Information

There's a lot of configuration you can do here. At a high level, these are the top level objects you can disable/enable for your needs:

* **Ingress**: If you don't require an Ingress resource, you can disable this using `ingress.enable: false`. If you do want to continue to use the Ingress, it is advised that you keep `service.https.type` set to `ClusterIP`. The Ingress is enabled by default, but does require some user provided values to work.
* **Service**: There are two services here, ldaps and https. You can disable either or both by using `service.<servic_type>.enable: false`. Both services can be set to type of `ClusterIP`, `LoadBalancer`, or `NodePort`. They are both enabled and set to `ClusterIP` by default.
* **Service Account**: If you decide you want (or don't want) a service account created to attach to the pod, you can control that with `serviceAccount.enable: false/true`. This is disabled by default.

There is an assumption made that you will be using TLS in this stack, either by generating a certificate through a ClusterIssuer/Issuer or providing a pre-created Secret containing two keys: TLS Key and TLS Certificate Chain.

The domain/origin URL of your Kanidm instance is controlled by `kanidm.domain` (required) and `kanidm.origin` (optional). If `kanidm.origin` is not specified, `kanidm.domain` is used in all instances where it would be referenced.

Nearly any resource can have annotations and labels specifically applied to it. Additionally, you can set `global.(labels|annotations)` to apply labels and/or annotations to ALL resources created by this chart.

For more specific configuration needs and further explanation of any values, please see the [full configuration values](#full-configuration-values) section.

### Data Persistence

There is no assumption made by this chart by default to give you any data persistence. The default behavior is to mount an `emptyDir: {}` volume at `/data` in the container. This means data will be lost if you restart the StatefulSet/Pod. If you would like to persist your Kanidm data (HIGHLY recommended), you have one of two options:

* Use `statefulset.storage.existingClaim` to specify the name of an existing PersistentVolumeClaim to bind to the pod. This requires you create this and the underlying PersistentVolume ahead of time however you would like.
* Use `statefulset.storage.volumeClaimTemplate` to use a PersistentVolumeClaimTemplate to generate a PVC and PV for you using the specified StorageClass. Be sure the StorageClass actually exists first.

VolumeClaimTemplate should follow standard format like the following (customize to your needs):

```yaml
volumeClaimTemplate:
  metadata:
    name: volume-name
  spec:
    accessModes: [ "ReadWriteOnce" ]
    resources:
      requests:
        storage: 10Gi
    storageClassName: your-storage-class
```

If you're struggling with file permissions in your persistence (for example, over NFS), look into NFS `mapall` and `mapgroup` options. You can also use `statefulset.podSecurityContext` to configure the user or group files are accessed as.

## Full Configuration Values

### Global

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| global.labels | object | {} | Labels that will be applied to all resources |
| global.annotations | object | {} | Annotations that will be applied to all resources |

### Kanidm Configuration (Top Level)

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| kanidm.domain | string | `""` | Domain to use for Kanidm |
| kanidm.origin | string | `""` | Origin to use for Kanidm (if different from domain) |

### Kanidm Configuration (Backups)

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| kanidm.backups.enabled | bool | true | Enable/disable database backups |
| kanidm.backups.path | string | /data/backups | Path to store backups |
| kanidm.backups.schedule | string | "00 00 * * *" | Schedule for backups |

### Kanidm Configuration (Services)

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| kanidm.https.enabled | bool | true | Enable/disable the HTTPS service |
| kanidm.https.port | int | 8443 | Port to use for the HTTPS service |
| kanidm.ldaps.enabled | bool | true | Enable/disable the LDAPS service |
| kanidm.ldaps.port | int | 3636 | Port to use for the LDAPS service |

### Kanidm Configuration (TLS)

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| kanidm.tls.externalCertificateSecret.enabled | bool | false | Enable/disable the use of an external certificate resource (mutually exclusive with `kanidm.tls.issuer`) |
| kanidm.tls.issuer | object | {} | Certificate issuer to use with ClusterIssuer/Issuer for cert generation (mutually exclusive with `kanidm.tls.externalCertificateSecret`) |
| kanidm.tls.hosts | list | [] | Hosts to use for the TLS certificate |
| kanidm.tls.secretName | string | "kanidm-tls" | Secret name to use for the TLS certificate |
| kanidm.tls.tlsKeySecretKeyName | string | "tls.key" | Secret key that holds the TLS certificate key |
| kanidm.tls.tlsChainSecretKeyName | string | "tls.crt" | Secret key that holds the TLS certificate chain |

### Service Account

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| serviceAccount.create | bool | false | Enable/disable creation of a dedicated Kubernetes Service Account |
| serviceAccount.name | string | "" | Name of the service account to use |

### Ingress

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| ingress.enabled | bool | true | Enable/disable the creation of an Ingress resource |
| ingress.ingressClassName | string | "" | IngressClass to use |
| ingress.annotations | object | {} | Annotations to apply to the Ingress resource |
| ingress.labels | object | {} | Labels to apply to the Ingress resource |
| ingress.host | string | "" | Hostname to use for the Ingress |

### Service

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| service.https.annotations | object | {} | Annotations to apply to the HTTPS Service resource |
| service.https.labels | object | {} | Labels to apply to the HTTPS Service resource |
| service.https.enabled | string | true | Enable/disable the HTTPS Service resource |
| service.https.type | string | ClusterIP | Service type to use for HTTPS (can be one of ClusterIP, NodePort, LoadBalancer) |
| service.https.port | int | 8443 | Port to use for the HTTPS Service resource |
| service.ldaps.annotations | object | {} | Annotations to apply to the LDAPS Service resource |
| service.ldaps.labels | object | {} | Labels to apply to the LDAPS Service resource |
| service.ldaps.enabled | string | true | Enable/disable the LDAPS Service resource |
| service.ldaps.type | string | ClusterIP | Service type to use for LDAPS (can be one of ClusterIP, NodePort, LoadBalancer) |
| service.ldaps.port | int | 3636 | Port to use for the LDAPS Service resource |

### StatefulSet

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| statefulset.annotations | object | {} | Annotations to apply to the StatefulSet resource |
| statefulset.labels | object | {} | Labels to apply to the StatefulSet resource |
| statefulset.podSecurityContext | object | {} | PodSecurityContext for StatefulSet object |
| statefulset.containerSecurityContext | object | {} | SecurityContext for the kanidm container |
| statefulset.imagePullSecrets | list | [] | Image pull secrets for use with private registries |
| statefulset.image.repository | string | kanidm/server | Image to use for the kanidm container (must be in format repostiory/image_name) |
| statefulset.image.tag | string | latest | Tag to use for the kanidm image |
| statefulset.image.pullPolicy | string | IfNotPresent | pullPolicy to use for kanidm images |
| statefulset.extraEnv | list | [] | Extra environment variables to pass to the kanidm container. Should be in `[{"name": name, "value": value}]` format |
| statefulset.livenessProbeEnabled | bool | true | Enable/disable liveness probe |
| statefulset.readinessProbeEnabled | bool | true | Enable/disable readiness probe |
| statefulset.extraPorts | list | [] | Extra ports to expose on the kanidm container. Should be in `[{"name": name, "containerPort": port}]` format |
| statefulset.resources.limits.cpu | string | 50M | CPU limit for the kanidm container |
| statefulset.resources.limits.memory | string | 100Mi | Memory limit for the kanidm container |
| statefulset.resources.requests.cpu | string | 10m | CPU request for the kanidm container |
| statefulset.resources.requests.memory | string | 50Mi | Memory request for the kanidm container |
| statefulset.storage.existingClaim | string | null | Existing PVC to use for the kanidm data volume |
| statefulset.storage.volumeClaimTemplate | string | {} | VolumeClaimTemplate to use for kanidm data PVC creation. See [#data-persistence](Data Persistence) for more information |

----------------------------------------------
Autogenerated from chart metadata using [helm-docs v1.14.2](https://github.com/norwoodj/helm-docs/releases/v1.14.2)

## Testing

To run the unit tests for this chart, you'll need to install the [Helm unittests plugin](https://github.com/helm-unittest/helm-unittest/) plugin and run the following command from this directory:

```shell
helm unittest .
```
