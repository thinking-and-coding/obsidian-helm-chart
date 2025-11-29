# Obsidian Helm Chart

A Helm chart for deploying [Obsidian](https://obsidian.md) note-taking application with web-based GUI on Kubernetes.

This chart deploys the [LinuxServer.io Obsidian Docker container](https://github.com/linuxserver/docker-obsidian) which uses the Selkies desktop streaming framework to provide a full-featured web-based desktop experience.

## Features

- Web-based Obsidian GUI accessible via browser (HTTP/HTTPS)
- Persistent storage for notes and configuration
- Configurable authentication (basic HTTP auth)
- Support for both HTTP and HTTPS access
- Minimal Selkies configuration with extensibility via extraEnv
- Resource limits and requests
- Health checks (liveness, readiness, startup probes)
- Ingress support for external access

## Prerequisites

- Kubernetes 1.19+
- Helm 3.0+
- PV provisioner support in the underlying infrastructure (if persistence is enabled)

## Installation

### Add the Helm chart repository (if available)

```bash
# If this chart is published to a Helm repository
helm repo add obsidian https://your-repo-url
helm repo update
```

### Install from local directory

```bash
# From the chart directory
helm install my-obsidian ./helm-chart

# With custom values
helm install my-obsidian ./helm-chart -f my-values.yaml

# With inline overrides
helm install my-obsidian ./helm-chart \
  --set persistence.size=20Gi \
  --set auth.enabled=true \
  --set auth.username=admin \
  --set auth.password=mypassword
```

## Uninstallation

```bash
helm uninstall my-obsidian
```

## Configuration

### Basic Configuration Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| `replicaCount` | Number of replicas (should be 1) | `1` |
| `image.repository` | Container image repository | `lscr.io/linuxserver/obsidian` |
| `image.tag` | Container image tag | `latest` |
| `image.pullPolicy` | Image pull policy | `IfNotPresent` |

### Persistence

| Parameter | Description | Default |
|-----------|-------------|---------|
| `persistence.enabled` | Enable persistent storage | `true` |
| `persistence.size` | PVC size | `10Gi` |
| `persistence.storageClass` | Storage class name | `""` (default) |
| `persistence.accessMode` | PVC access mode | `ReadWriteOnce` |
| `persistence.existingClaim` | Use existing PVC | `""` |

### Service

| Parameter | Description | Default |
|-----------|-------------|---------|
| `service.type` | Service type | `ClusterIP` |
| `service.httpPort` | HTTP port (limited functionality) | `3000` |
| `service.httpsPort` | HTTPS port (recommended) | `3001` |

### Ingress

| Parameter | Description | Default |
|-----------|-------------|---------|
| `ingress.enabled` | Enable ingress | `false` |
| `ingress.className` | Ingress class name | `""` |
| `ingress.annotations` | Ingress annotations | `{}` |
| `ingress.hosts` | Ingress hosts configuration | See values.yaml |
| `ingress.tls` | Ingress TLS configuration | `[]` |

### Environment Variables

| Parameter | Description | Default |
|-----------|-------------|---------|
| `env.puid` | User ID for file ownership | `"1000"` |
| `env.pgid` | Group ID for file ownership | `"1000"` |
| `env.timezone` | Timezone setting | `"Etc/UTC"` |
| `env.title` | Application title | `"Obsidian"` |

### Authentication

| Parameter | Description | Default |
|-----------|-------------|---------|
| `auth.enabled` | Enable basic HTTP auth | `false` |
| `auth.username` | Basic auth username | `""` |
| `auth.password` | Basic auth password | `""` |

### Selkies Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `selkies.lcAll` | Language/locale setting | `""` |
| `selkies.subfolder` | Subfolder for reverse proxy | `""` |
| `selkies.fileManagerPath` | Custom file manager path | `""` |
| `selkies.encoder` | Video encoder selection | `"x264enc,x264enc-striped,jpeg"` |
| `selkies.framerate` | Framerate range or fixed value | `"8-120"` |
| `selkies.uiTitle` | UI title in sidebar | `"Selkies"` |
| `selkies.uiShowLogo` | Show Selkies logo | `true` |
| `selkies.uiShowSidebar` | Show main sidebar | `true` |

### Advanced Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `shmSize` | Shared memory size (required for Electron) | `1Gi` |
| `resources` | CPU/memory resource requests/limits | `{}` |
| `nodeSelector` | Node labels for pod assignment | `{}` |
| `tolerations` | Tolerations for pod assignment | `[]` |
| `affinity` | Affinity rules for pod assignment | `{}` |
| `extraEnv` | Additional environment variables | `[]` |

## Usage Examples

### Basic Installation with Authentication

```bash
helm install obsidian ./helm-chart \
  --set auth.enabled=true \
  --set auth.username=myuser \
  --set auth.password=mypassword
```

### Installation with Ingress

```bash
helm install obsidian ./helm-chart \
  --set ingress.enabled=true \
  --set ingress.className=nginx \
  --set ingress.hosts[0].host=obsidian.example.com \
  --set ingress.hosts[0].paths[0].path=/ \
  --set ingress.hosts[0].paths[0].pathType=Prefix \
  --set ingress.hosts[0].paths[0].port=3001
```

### Installation with Larger Storage

```bash
helm install obsidian ./helm-chart \
  --set persistence.size=50Gi \
  --set persistence.storageClass=fast-ssd
```

### Installation with Custom Selkies Variables

```bash
helm install obsidian ./helm-chart \
  --set extraEnv[0].name=SELKIES_AUDIO_ENABLED \
  --set extraEnv[0].value="false" \
  --set extraEnv[1].name=SELKIES_FRAMERATE \
  --set extraEnv[1].value="30"
```

### Installation with Chinese Language Support

```bash
helm install obsidian ./helm-chart \
  --set selkies.lcAll=zh_CN.UTF-8
```

## Important Notes

### HTTPS Requirement

**HTTPS (port 3001) is REQUIRED for full functionality.** Modern browser features such as WebCodecs, used for video and audio, will not function over an insecure HTTP connection. The container uses a self-signed certificate by default.

### Security Considerations

- The container provides privileged access to a desktop environment
- Default configuration has no authentication (use `auth.enabled=true` for basic auth)
- For production/internet exposure, use a reverse proxy with robust authentication
- The web interface includes a terminal with passwordless sudo access

### Shared Memory Size

The `shmSize` parameter (default: 1Gi) is required for Electron applications to function properly. This is equivalent to Docker's `--shm-size` parameter.

### User/Group IDs

The `env.puid` and `env.pgid` parameters control file ownership in the container. Ensure these match your desired user/group for the mounted volumes.

## Advanced Selkies Configuration

This chart includes minimal Selkies environment variables. For advanced configuration, use the `extraEnv` parameter to add additional Selkies variables. Full list of available variables:

- UI customization (50+ `SELKIES_UI_*` variables)
- Video streaming options (`SELKIES_ENCODER`, `SELKIES_FRAMERATE`, etc.)
- Audio configuration (`SELKIES_AUDIO_*` variables)
- Hardening options (`HARDEN_DESKTOP`, `DISABLE_SUDO`, etc.)
- GPU support (`DRINODE`, `DRI_NODE` for hardware acceleration)

See the [LinuxServer.io Selkies baseimage documentation](https://github.com/linuxserver/docker-baseimage-selkies) for complete details.

## Troubleshooting

### Pod not starting

Check the pod logs:
```bash
kubectl logs -f pod/obsidian-xxxxx
```

Verify the PVC is bound:
```bash
kubectl get pvc
```

### Can't access the application

Verify the service is running:
```bash
kubectl get svc
```

Check if the pod is ready:
```bash
kubectl get pods
```

For ClusterIP service, use port-forward:
```bash
kubectl port-forward svc/obsidian 3001:3001
```

### Self-signed certificate warnings

The container uses a self-signed certificate by default. You can:
1. Accept the certificate in your browser
2. Configure Ingress with a valid TLS certificate
3. Use a reverse proxy to handle TLS termination

## Links

- [Obsidian Website](https://obsidian.md)
- [LinuxServer.io Docker Image](https://github.com/linuxserver/docker-obsidian)
- [Selkies Baseimage](https://github.com/linuxserver/docker-baseimage-selkies)
- [Obsidian Releases](https://github.com/obsidianmd/obsidian-releases)

## License

This Helm chart is provided as-is. The Obsidian application and LinuxServer.io Docker image have their own respective licenses.
