
# ms-base Helm Chart

This chart provides the base configuration and best practices for deploying microservices in Kubernetes using the CodeDesignPlus SDK. It includes support for deployment, service configuration, probes, autoscaling, Vault integration, Istio, and more.

## Structure

- **deployment.yaml**: Main deployment template.
- **service.yaml**: Kubernetes service configuration.
- **ingress.yaml**: Optional Ingress configuration.
- **vaultconfig.yaml / vaultsecret.yaml**: HashiCorp Vault integration.
- **virtualservices.yaml**: Istio VirtualService integration.
- **values.yaml**: Default and configurable values.

## Usage

1. Add the chart as a dependency or copy it to your charts repository.
2. Customize the values in `values.yaml` according to your microservice needs.
3. Deploy using Helm:

```sh
helm install <release-name> ./ms-base -f values.yaml
```

## Configurable Values

All configurable values available in `values.yaml`:

| Key | Description | Example/Values |
|-----|-------------|----------------|
| `environment` | Deployment environment | Staging, Production |
| `replicaCount` | Number of replicas | 1 |
| `image.repository` | Container image | nginx |
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

See `values.yaml` and `values.schema.yaml` for advanced examples and details.

## Integrations

- **Vault**: Secret management using HashiCorp Vault.
- **Istio**: Support for VirtualService and gateways.
- **Probes**: Configurable liveness and readiness probes.
- **Autoscaling**: Optional HPA configuration.

## Example Helm Command

```sh
helm upgrade --install ms-catalogs-rest codedesignplus/ms-catalogs-rest \ 
    --namespace codedesignplus \ 
    --set ms-base.vault.token=******** \ 
    --create-namespace
```

## Example Custom Values

```yaml
environment: Staging
replicaCount: 2
image:
  repository: myrepo/myservice
  pullPolicy: IfNotPresent
  tag: v1.0.0
service:
  type: ClusterIP
  ports:
    - name: rest
      port: 5000
      targetPort: rest
      protocol: TCP
    - name: grpc
      port: 5001
      targetPort: grpc
      protocol: TCP
ingress:
  enabled: true
  className: "nginx"
  hosts:
    - host: myservice.local
      paths:
        - path: /
          pathType: ImplementationSpecific
vault:
  create: true
  token: "<vault-token>"
  server: "https://vault.mydomain.com"
virtualService:
  create: true
  hosts:
    - "myservice.local"
  gateways:
    - istio-ingress/istio-gateway
```
## Requirements

- Kubernetes >= 1.20
- Helm >= 3.0
- Istio (Required, for VirtualService)

## License
This project is licensed under the LGPL-3.0 License - see the [LICENSE](LICENSE.md) file for details
- Vault (required for secret management)

## License

For more information, please refer to the [LICENSE](./LICENSE.md) file in this repository.
