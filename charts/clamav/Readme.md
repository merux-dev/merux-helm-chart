# ClamAV Helm Chart

This repository contains a Helm chart to deploy [ClamAV](https://www.clamav.net/) on a Kubernetes cluster. ClamAV is an open-source antivirus engine for detecting trojans, viruses, malware, and other malicious threats.
It can be connected to a Nextcloud instance by using the service name as host and selecting `ClamAV Daemon` as your mode.

## Prerequisites

- Kubernetes 1.16+
- Helm 3.0+

## Getting Started

### 1. Install the Chart

To install the chart with the release name `my-clamav` in the `security` namespace:

```bash
helm install my-clamav ./clamav --namespace security --create-namespace
```

### 2. Uninstall the Chart

To uninstall/delete the `my-clamav` deployment:

```bash
helm uninstall my-clamav --namespace security
```

The command removes all the Kubernetes components associated with the chart and deletes the release.

## Configuration

The following table lists the configurable parameters of the ClamAV chart and their default values. Users can override these defaults by modifying the `values.yaml` file or by using `--set` flags during `helm install`.

| Parameter                  | Description                                     | Default                                 |
| -------------------------- | ----------------------------------------------- | --------------------------------------- |
| `replicaCount`             | Number of ClamAV replicas                       | `1`                                     |
| `image.repository`         | ClamAV image repository                         | `clamav/clamav`                         |
| `image.tag`                | ClamAV image tag (version)                      | `"latest"`                              |
| `image.pullPolicy`         | Image pull policy                               | `IfNotPresent`                          |
| `service.type`             | Kubernetes Service type                         | `ClusterIP`                             |
| `service.port`             | Port exposed by the Service & Container         | `3310`                                  |
| `env.noFreshclamd`         | Whether to disable automatic virus definition updates | `"false"`                         |
| `resources.requests.cpu`   | CPU requests for the ClamAV container           | `"500m"`                                |
| `resources.requests.memory`| Memory requests for the ClamAV container        | `"1Gi"`                                 |
| `resources.limits.cpu`     | CPU limits for the ClamAV container             | `"1000m"`                               |
| `resources.limits.memory`  | Memory limits for the ClamAV container          | `"2Gi"`                                 |
| `nameOverride`             | String to partially override the chart name     | `""`                                    |
| `fullnameOverride`         | String to fully override the chart name         | `""`                                    |

### Example: Customizing Installation

You can dynamically adjust limits, tags, or ports to match your specific production requirements using `--set`:

```bash
helm install my-clamav ./clamav \
  --namespace security \
  --set image.tag="1.0.1" \
  --set service.port=3311 \
  --set resources.limits.memory="4Gi" \
  --set resources.requests.memory="2Gi"
```

> **Note:** ClamAV is memory-heavy. It is highly recommended not to skip or set the memory limits too low.