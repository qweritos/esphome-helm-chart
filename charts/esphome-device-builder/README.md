# ESPHome Device Builder

This chart deploys [ESPHome Device Builder](https://github.com/esphome/device-builder) on Kubernetes using the official `ghcr.io/esphome/esphome` image. Device Builder stores ESPHome YAML files and build state under `/config` and serves the web UI on port `6052`.

## Install

```sh
helm install esphome-device-builder oci://registry.andrey.wtf/charts/esphome-device-builder
```

Expose it with ingress:

```yaml
ingress:
  enabled: true
  className: nginx
  hosts:
    - host: esphome.example.com
      paths:
        - path: /
          pathType: Prefix
dashboard:
  trustedDomains:
    - esphome.example.com
auth:
  enabled: true
  username: admin
  password: change-me
```

Use host networking for LAN discovery / OTA scenarios when your cluster networking does not pass mDNS or direct device traffic cleanly:

```yaml
hostNetwork:
  enabled: true
service:
  enabled: false
```

## Values

| Value | Default | Description |
| --- | --- | --- |
| `replicaCount` | `1` | Number of pods. Keep `1` with `ReadWriteOnce` storage. |
| `strategy` | `RollingUpdate` | Deployment update strategy. Use `Recreate` for single-node host-network deployments. |
| `image.repository` | `ghcr.io/esphome/esphome` | Container image repository. |
| `image.tag` | `""` | Image tag. Empty uses `Chart.appVersion`. |
| `image.pullPolicy` | `IfNotPresent` | Kubernetes image pull policy. |
| `imagePullSecrets` | `[]` | Image pull secret references. |
| `nameOverride` | `""` | Override chart name. |
| `fullnameOverride` | `""` | Override generated resource names. |
| `serviceAccount.create` | `true` | Create a service account. |
| `serviceAccount.automount` | `false` | Automount service account token. |
| `serviceAccount.annotations` | `{}` | Service account annotations. |
| `serviceAccount.name` | `""` | Existing service account name when not creating one. |
| `podAnnotations` | `{}` | Pod annotations. |
| `podLabels` | `{}` | Extra pod labels. |
| `podSecurityContext` | `{}` | Pod security context. |
| `securityContext` | see `values.yaml` | Container security context. Set `privileged: true` here if USB/device access requires it. |
| `hostNetwork.enabled` | `false` | Run the pod on the node network. Useful for mDNS and OTA workflows. |
| `hostNetwork.dnsPolicy` | `ClusterFirstWithHostNet` | DNS policy used with host networking. |
| `service.enabled` | `true` | Create a Service. |
| `service.type` | `ClusterIP` | Service type. |
| `service.port` | `6052` | Service HTTP port. |
| `service.targetPort` | `http` | Service target port. |
| `service.nodePort` | `null` | NodePort value when `service.type=NodePort`. |
| `service.externalTrafficPolicy` | `""` | External traffic policy for NodePort/LoadBalancer. |
| `service.loadBalancerIP` | `""` | Static LoadBalancer IP. |
| `service.loadBalancerSourceRanges` | `[]` | LoadBalancer source ranges. |
| `service.annotations` | `{}` | Service annotations. |
| `service.labels` | `{}` | Extra Service labels. |
| `ingress.enabled` | `false` | Create an Ingress. |
| `ingress.className` | `""` | IngressClass name. |
| `ingress.annotations` | `{}` | Ingress annotations. Ensure WebSocket upgrade headers are supported by your controller. |
| `ingress.hosts` | `esphome.example.local` | Ingress hosts and paths. |
| `ingress.tls` | `[]` | Ingress TLS entries. |
| `dashboard.configPath` | `/config` | ESPHome configuration directory inside the container. |
| `dashboard.port` | `6052` | Dashboard listen port passed to the container command. |
| `dashboard.trustedDomains` | `[]` | Sets `ESPHOME_TRUSTED_DOMAINS` for reverse proxies that rewrite `Host`. |
| `dashboard.extraArgs` | `[]` | Extra CLI args appended after `dashboard <configPath> --port <port>`. |
| `auth.enabled` | `false` | Enable `ESPHOME_USERNAME` / `ESPHOME_PASSWORD` authentication. |
| `auth.username` | `""` | Username used when the chart creates the auth secret. |
| `auth.password` | `""` | Password used when the chart creates the auth secret. Prefer an external secret for shared values files. |
| `auth.existingSecret` | `""` | Existing secret containing auth keys. |
| `auth.usernameKey` | `username` | Username key in the auth secret. |
| `auth.passwordKey` | `password` | Password key in the auth secret. |
| `remoteBuild.enabled` | `false` | Adds the remote-build peer-link container port and env vars. The feature is enabled from Device Builder settings. |
| `remoteBuild.port` | `6055` | Remote-build container port and `ESPHOME_REMOTE_BUILD_PORT`. |
| `remoteBuild.host` | `0.0.0.0` | `ESPHOME_REMOTE_BUILD_HOST`. |
| `remoteBuild.service.enabled` | `true` | Add the remote-build port to the Service. |
| `remoteBuild.service.port` | `6055` | Service port for remote-build. |
| `persistence.enabled` | `true` | Persist `/config` with a PVC. Uses `emptyDir` when false. |
| `persistence.existingClaim` | `""` | Existing PVC name. |
| `persistence.storageClass` | `""` | StorageClass name. Empty uses the cluster default. |
| `persistence.accessModes` | `[ReadWriteOnce]` | PVC access modes. |
| `persistence.size` | `5Gi` | PVC requested size. |
| `persistence.annotations` | `{}` | PVC annotations. |
| `persistence.labels` | `{}` | Extra PVC labels. |
| `env.TZ` | `""` | Optional timezone environment variable. |
| `extraEnv` | `[]` | Additional container environment variables. |
| `extraEnvFrom` | `[]` | Additional `envFrom` entries. |
| `extraVolumes` | `[]` | Additional pod volumes. |
| `extraVolumeMounts` | `[]` | Additional container volume mounts. |
| `initContainers` | `[]` | Additional init containers. |
| `sidecars` | `[]` | Additional sidecar containers. |
| `resources` | `{}` | Container resource requests and limits. |
| `livenessProbe` | enabled | Liveness probe configuration. |
| `readinessProbe` | enabled | Readiness probe configuration. |
| `startupProbe` | disabled | Startup probe configuration. |
| `nodeSelector` | `{}` | Pod node selector. |
| `tolerations` | `[]` | Pod tolerations. |
| `affinity` | `{}` | Pod affinity. |
| `topologySpreadConstraints` | `[]` | Pod topology spread constraints. |
| `priorityClassName` | `""` | Pod priority class. |
| `runtimeClassName` | `""` | Pod runtime class. |
| `schedulerName` | `""` | Pod scheduler name. |
| `terminationGracePeriodSeconds` | `30` | Pod termination grace period. |

## Notes

Device Builder warns when exposed without authentication. For ingress or LoadBalancer deployments, enable `auth` or protect access at the edge.

When using ingress behind a reverse proxy that changes the upstream `Host`, set `dashboard.trustedDomains` to the public hostnames so browser WebSocket checks pass.
