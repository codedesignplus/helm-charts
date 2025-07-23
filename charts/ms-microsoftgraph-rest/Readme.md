# ms-microsoftgraph-rest Helm Chart

This chart provides the configuration for deploying the ms-microsoftgraph-rest microservice in Kubernetes, based on the CodeDesignPlus SDK. It leverages the ms-base chart for best practices and common features such as deployment, service configuration, probes, autoscaling, Vault integration, Istio, and more.

## Structure

- **Chart.yaml**: Chart metadata.
- **values.yaml**: Default and configurable values for ms-microsoftgraph-rest.

## Usage

1. Add the chart as a dependency or copy it to your charts repository.
2. Customize the values in `values.yaml` according to your microservice needs.
3. Deploy using Helm:

```sh
helm upgrade --install ms-microsoftgraph-rest codedesignplus/ms-microsoftgraph-rest \
    --namespace <namespace> \
    --set ms-base.vault.token=<vault-token> \
    --set ms-base.virtualService.http[0].route[0].destination.host=ms-microsoftgraph-rest.<namespace>.svc.cluster.local \
    --create-namespace
```

## Configurable Values

All configurable values available in `values.yaml` (inherited from ms-base):

| Key | Description | Example/Values |
|-----|-------------|----------------|
| `environment` | Deployment environment | Staging, Production |
| `replicaCount` | Number of replicas | 1 |
| `image.repository` | Container image | codedesignplus/ms-microsoftgraph-rest |
| `image.pullPolicy` | Image pull policy | IfNotPresent, Always, Never |
| `image.tag` | Image tag | latest |
| `imagePullSecrets` | Secrets for pulling private images | [] |
| `nameOverride` | Override chart name | "" |
| `fullnameOverride` | Override full name | "" |
| `serviceAccount.create` | Create ServiceAccount | true/false |
| `serviceAccount.automount` | Automatically mount credentials | true/false |
| `serviceAccount.annotations` | ServiceAccount annotations | {} |
| `serviceAccount.name` | ServiceAccount name | "" |
| `podAnnotations` | Pod annotations | {} |
| `podLabels` | Pod labels | {} |
| `podSecurityContext` | Pod security context | {} |
| `securityContext` | Container security context | {} |
| `service.type` | Service type | ClusterIP, NodePort, LoadBalancer |
| `service.ports` | List of exposed ports | [{name, port, targetPort, protocol}] |
| `ingress.enabled` | Enable Ingress | true/false |
| `ingress.className` | Ingress class | "" |
| `ingress.annotations` | Ingress annotations | {} |
| `ingress.hosts` | Ingress hosts and paths | [{host, paths}] |
| `ingress.tls` | Ingress TLS configuration | [{secretName, hosts}] |
| `resources` | Resource limits and requests | {} |
| `livenessProbe.httpGet.path` | Liveness probe path | /health/live |
| `livenessProbe.httpGet.port` | Liveness probe port | http |
| `readinessProbe.httpGet.path` | Readiness probe path | /health/ready |
| `readinessProbe.httpGet.port` | Readiness probe port | http |
| `autoscaling.enabled` | Enable autoscaling | true/false |
| `autoscaling.minReplicas` | Minimum replicas | 1 |
| `autoscaling.maxReplicas` | Maximum replicas | 100 |
| `autoscaling.targetCPUUtilizationPercentage` | CPU percentage for scaling | 80 |
| `volumes` | Additional volumes | [] |
| `volumeMounts` | Additional volume mounts | [] |
| `nodeSelector` | Node selector | {} |
| `tolerations` | Tolerations | [] |
| `affinity` | Affinity | {} |
| `virtualService.create` | Enable Istio VirtualService | true/false |
| `virtualService.hosts` | VirtualService hosts | ["chart-example.local"] |
| `virtualService.gateways` | VirtualService gateways | ["istio-ingress/services-gateway"] |
| `virtualService.http` | VirtualService HTTP configuration | [{route: [{destination: {host, port: {number}}}]}] |
| `vault.create` | Enable Vault integration | true/false |
| `vault.token` | Vault token | The token for Vault authentication. |
| `vault.server` | Vault server URL | The Vault server address. |
| `vault.solution` | Vault solution | The Vault solution to connect to secret (key-value store, database, rabbitmq or transit). |

See `ms-base/values.yaml` and `ms-base/values.schema.yaml` for advanced examples and details.

## Integrations

- **Vault**: Secret management using HashiCorp Vault.
- **Istio**: Support for VirtualService and gateways.
- **Probes**: Configurable liveness and readiness probes.
- **Autoscaling**: Optional HPA configuration.

## Example Custom Values

```yaml
ms-base:
  environment: Staging
  replicaCount: 1

  image:
    repository: codedesignplus/ms-microsoftgraph-rest
    pullPolicy: IfNotPresent
    tag: "latest"

  imagePullSecrets: []
  fullnameOverride: "ms-microsoftgraph-rest"
  nameOverride: "ms-microsoftgraph-rest"

  service:
    type: ClusterIP
    ports:
      - name: http
        port: 5000
        targetPort: http
        protocol: TCP

  resources: 
    limits:
      cpu: 100m
      memory: 128Mi
    requests:
      cpu: 100m
      memory: 128Mi

  autoscaling:
    enabled: true
    minReplicas: 1
    maxReplicas: 5
    targetMemoryUtilizationPercentage: 80

  virtualService: 
    create: true
    namespace: istio-ingress
    hosts:
      - services.codedesignplus.com
    gateways:
      - istio-ingress/istio-gateway
    http:
    - name: ms-microsoftgraph
      match:
      - uri:
          prefix: /ms-microsoftgraph/
      rewrite:
        uri: /
      route:
      - destination:
          host: ms-microsoftgraph-rest.codedesignplus.svc.cluster.local
          port:
            number: 5000
    
  vault:
    server: http://vault-internal.vault-operator.svc.cluster.local:8200
    solution: security-codedesignplus
```
## Requirements

- Kubernetes >= 1.20
- Helm >= 3.0
- Istio (Required, for VirtualService)

## License
This project is licensed under the LGPL-3.0 License - see the [LICENSE](LICENSE.md) file for details
