# Homebridge Helm Chart

This Helm chart deploys [Homebridge](https://homebridge.io/) on a Kubernetes cluster, providing HomeKit support for smart home devices.

## Features

- Deploys Homebridge with the official Docker image
- Web-based configuration interface
- Persistent storage for configuration and plugins
- Automatic initialization of config.json on first install
- Ingress support for external access
- Configurable resource limits
- Support for custom environment variables

## Prerequisites

- Kubernetes 1.19+
- Helm 3.0+
- PersistentVolume provisioner support in the underlying infrastructure (if persistence is enabled)

## Installing the Chart

To install the chart with the release name `homebridge`:

```bash
helm install homebridge ./homebridge
```

To install with custom values:

```bash
helm install homebridge ./homebridge -f custom-values.yaml
```

## Uninstalling the Chart

To uninstall the `homebridge` deployment:

```bash
helm uninstall homebridge
```

This command removes all the Kubernetes components associated with the chart and deletes the release.

## Configuration

The following table lists the configurable parameters of the Homebridge chart and their default values.

### General Settings

| Parameter | Description | Default |
|-----------|-------------|---------|
| `replicaCount` | Number of Homebridge replicas | `1` |
| `image.repository` | Homebridge image repository | `homebridge/homebridge` |
| `image.pullPolicy` | Image pull policy | `IfNotPresent` |
| `image.tag` | Homebridge image tag | `latest` |
| `nameOverride` | Override chart name | `""` |
| `fullnameOverride` | Override full chart name | `""` |

### Service Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `service.type` | Kubernetes service type | `ClusterIP` |
| `service.port` | Web interface port | `8581` |

### Ingress Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `ingress.enabled` | Enable ingress controller resource | `true` |
| `ingress.className` | Ingress class name | `""` |
| `ingress.annotations` | Ingress annotations | `{}` |
| `ingress.hosts[0].host` | Hostname for the ingress | `homebridge.local` |
| `ingress.hosts[0].paths[0].path` | Path for the ingress | `/` |
| `ingress.hosts[0].paths[0].pathType` | Path type | `Prefix` |
| `ingress.tls` | TLS configuration | `[]` |

### Persistence Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `persistence.enabled` | Enable persistent storage | `true` |
| `persistence.storageClass` | Storage class name | `""` |
| `persistence.accessMode` | Access mode | `ReadWriteOnce` |
| `persistence.size` | Storage size | `1Gi` |
| `persistence.existingClaim` | Use existing PVC | `""` |

### Initial Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `initialConfig.enabled` | Enable automatic config initialization | `true` |
| `initialConfig.config` | Initial config.json content | See values.yaml |

### Resource Management

| Parameter | Description | Default |
|-----------|-------------|---------|
| `resources.limits.cpu` | CPU limit | Not set |
| `resources.limits.memory` | Memory limit | Not set |
| `resources.requests.cpu` | CPU request | Not set |
| `resources.requests.memory` | Memory request | Not set |

### Other Settings

| Parameter | Description | Default |
|-----------|-------------|---------|
| `env` | Additional environment variables | `[]` |
| `hostNetwork` | Use host network mode | `false` |
| `nodeSelector` | Node labels for pod assignment | `{}` |
| `tolerations` | Tolerations for pod assignment | `[]` |
| `affinity` | Affinity for pod assignment | `{}` |

## Example Configurations

### Basic Installation with Custom Hostname

```yaml
ingress:
  enabled: true
  hosts:
    - host: homebridge.example.com
      paths:
        - path: /
          pathType: Prefix
  tls:
    - secretName: homebridge-tls
      hosts:
        - homebridge.example.com
```

### With Custom Storage Class and Size

```yaml
persistence:
  enabled: true
  storageClass: "fast-ssd"
  size: 5Gi
```

### With Resource Limits

```yaml
resources:
  limits:
    cpu: 500m
    memory: 512Mi
  requests:
    cpu: 250m
    memory: 256Mi
```

### With Custom Initial Configuration

```yaml
initialConfig:
  enabled: true
  config: |
    {
      "bridge": {
        "name": "My Homebridge",
        "username": "AA:BB:CC:DD:EE:FF",
        "port": 51826,
        "pin": "123-45-678"
      },
      "accessories": [],
      "platforms": [
        {
          "name": "Config",
          "port": 8581,
          "platform": "config"
        }
      ]
    }
```

### With Environment Variables

```yaml
env:
  - name: TZ
    value: "America/New_York"
  - name: HOMEBRIDGE_CONFIG_UI
    value: "1"
```

## Accessing Homebridge

After installation, you can access the Homebridge web interface:

1. If using Ingress (default):
   - Access via the configured hostname (e.g., `http://homebridge.local`)

2. If using port-forward:
   ```bash
   kubectl port-forward svc/homebridge 8581:8581
   ```
   Then visit `http://localhost:8581`

Default credentials:
- Username: `admin`
- Password: `admin`

**Important**: Change these credentials after your first login!

## HomeKit Pairing

To pair with HomeKit:
1. Open the Home app on your iOS device
2. Tap the "+" icon to add a new accessory
3. Scan the QR code shown in the Homebridge UI, or enter the PIN code manually
4. Default PIN: `031-45-154` (can be customized in values.yaml)

## Troubleshooting

### Pod is not starting

Check the pod logs:
```bash
kubectl logs -f deployment/homebridge
```

### Persistence issues

Verify the PVC status:
```bash
kubectl get pvc
```

### Configuration not initializing

Check if the ConfigMap exists:
```bash
kubectl get configmap
```

View the init container logs:
```bash
kubectl logs homebridge-pod-name -c init-config
```

## Links

- [Homebridge Documentation](https://github.com/homebridge/homebridge/wiki)
- [Homebridge Docker](https://github.com/homebridge/homebridge/wiki/Install-Homebridge-on-Docker)
- [Plugin Registry](https://www.npmjs.com/search?q=homebridge-plugin)
