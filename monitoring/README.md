# Grafana Observability Stack — Self-Hosted en MicroK8s

## Arquitectura

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         GRAFANA (UI + Alerting)                          │
│   Dashboards, Explore, Service Map, SLOs, Drilldown, Correlación        │
│   https://services.kappali.com/grafana                                  │
└──────────┬──────────────────────┬──────────────────────┬────────────────┘
           │                      │                      │
     ┌─────▼─────┐         ┌─────▼─────┐         ┌─────▼─────┐
     │ Prometheus │         │   Loki    │         │   Tempo   │
     │ (métricas) │         │  (logs)   │         │ (traces)  │
     └─────▲─────┘         └─────▲─────┘         └─────▲─────┘
           │                      │                      │
           └──────────────────────┼──────────────────────┘
                                  │
                    ┌─────────────▼─────────────┐
                    │       GRAFANA ALLOY        │
                    │  (reemplaza OTel Collector) │
                    │                            │
                    │  • OTLP receiver (:4317)   │
                    │  • Faro receiver (:12347)  │
                    │  • Batch processor         │
                    │  • Export → Tempo/Loki/Pro │
                    └─────────────▲─────────────┘
                                  │
           ┌──────────────────────┼──────────────────────┐
           │                      │                      │
    ┌──────┴──────┐      ┌───────┴───────┐     ┌───────┴───────┐
    │ 73 Microsvcs│      │ Kappali.Front │     │   Pyroscope   │
    │ (.NET OTel) │      │ (Faro SDK)    │     │  (profiling)  │
    └─────────────┘      └───────────────┘     └───────────────┘
```

## Componentes Instalados

| Componente | Chart | Namespace | Puerto | Función |
|---|---|---|---|---|
| **Grafana** | `grafana/grafana` | monitoring | 80 | UI, dashboards, alerting, Explore |
| **Alloy** | `grafana/alloy` | monitoring | 4317, 4318, 12347 | Collector universal (OTLP + Faro + logs) |
| **Tempo** | `grafana/tempo` | monitoring | 3200, 4317 | Almacén de traces distribuidos |
| **Loki** | `grafana/loki` | monitoring | 3100 (gateway:80) | Almacén de logs centralizados |
| **Prometheus** | `prometheus-community/prometheus` | monitoring | 80 | Almacén de métricas |
| **Pyroscope** | `grafana/pyroscope` | monitoring | 4040 | Continuous profiling (flamegraphs) |
| **k6-operator** | `grafana/k6-operator` | monitoring | - | Load testing distribuido en cluster |

## Acceso

| URL | Credenciales |
|---|---|
| `https://services.kappali.com/grafana` | admin / K4pp4l1-Gr4f4n4! |
| `https://services.kappali.com/alloy/collect` | Faro endpoint (frontend RUM) |

## Datasources Configurados en Grafana

| Datasource | URL Interna | UID |
|---|---|---|
| Prometheus | `http://prometheus-server.monitoring.svc.cluster.local` | prometheus |
| Loki | `http://loki-gateway.monitoring.svc.cluster.local` | loki |
| Tempo | `http://tempo.monitoring.svc.cluster.local:3200` | tempo |
| Pyroscope | `http://pyroscope.monitoring.svc.cluster.local:4040` | pyroscope |

## Correlación entre Señales

- **Tempo → Loki**: Traces a logs via `trace_id`
- **Tempo → Prometheus**: Traces a métricas via span metrics
- **Tempo → Pyroscope**: Traces a profiles
- **Tempo → Service Graph**: Mapa de dependencias entre microservicios

## Métricas Generadas por Tempo (metrics_generator)

Tempo genera automáticamente métricas RED a partir de los traces:

| Métrica | Tipo | Descripción |
|---|---|---|
| `traces_service_graph_request_total` | counter | Requests entre servicios (Service Map) |
| `traces_service_graph_request_server_seconds_*` | histogram | Latencia server-side |
| `traces_service_graph_request_client_seconds_*` | histogram | Latencia client-side |
| `traces_spanmetrics_calls_total` | counter | Total de calls por servicio/operación |
| `traces_spanmetrics_latency_*` | histogram | Latencia por operación |
| `traces_spanmetrics_size_total` | counter | Tamaño de spans |

Processors activos: `service-graphs`, `span-metrics`, `local-blocks`

## Alloy — Configuración del Collector

Alloy reemplaza el anterior OpenTelemetry Collector (`excellenceforge-opentelemetry-collector`). Actúa como punto único de ingesta para toda la telemetría:

### Receivers
- **OTLP gRPC** (`:4317`) — Microservicios .NET envían traces, métricas y logs
- **OTLP HTTP** (`:4318`) — Alternativa HTTP
- **Faro** (`:12347`) — Frontend Nuxt envía Web Vitals, errores JS, traces de navegación

### Exporters
- **Traces** → Tempo via OTLP (`tempo.monitoring:4317`)
- **Logs** → Loki via HTTP (`loki-gateway.monitoring:80`)
- **Métricas** → Prometheus via remote_write (`prometheus-server.monitoring:80`)

### Migración Transparente

Los microservicios apuntan a `alloy.monitoring.svc.cluster.local:4317` via las variables:
```yaml
LOGGER__OTELENDPOINT: http://alloy.monitoring.svc.cluster.local:4317
OBSERVABILITY__SERVEROTEL: http://alloy.monitoring.svc.cluster.local:4317
```

También existe un ExternalName Service en el namespace `otel-collector` para backwards compatibility:
```yaml
excellenceforge-opentelemetry-collector.otel-collector.svc → alloy.monitoring.svc
```

## Profiling (Pyroscope)

Los microservicios tienen pod annotations para que Pyroscope los descubra automáticamente:

```yaml
podAnnotations:
  profiles.grafana.com/memory.scrape: "true"
  profiles.grafana.com/memory.port: "5000"
  profiles.grafana.com/cpu.scrape: "true"
  profiles.grafana.com/cpu.port: "5000"
```

Estas annotations se definen en `helm-charts/charts/ms-base/values.yaml` y se heredan a todos los microservicios.

## RUM Frontend (Faro SDK)

El frontend Nuxt (`Kappali.Frontend`) está instrumentado con Grafana Faro:

- **Plugin**: `app/plugins/faro.client.ts`
- **Paquetes**: `@grafana/faro-web-sdk`, `@grafana/faro-web-tracing`
- **Endpoint**: `https://services.kappali.com/alloy/collect` → Alloy Faro receiver
- **Datos capturados**: Web Vitals (LCP, CLS, INP), errores JS, traces de navegación, sessions

## Load Testing (k6-operator)

El k6-operator permite ejecutar tests de carga dentro del cluster como CRDs de Kubernetes.

### Ejemplo de uso:
```bash
kubectl apply -f k6-example-configmap.yaml   # Script k6
kubectl apply -f k6-example-testrun.yaml     # Ejecutar test
kubectl get testrun -n monitoring -w          # Ver progreso
```

Los resultados se envían a Prometheus via remote_write y se visualizan en Grafana.

## Archivos de Values

| Archivo | Componente | Notas |
|---|---|---|
| `values-grafana.yaml` | Grafana | Datasources, dashboards, persistence, subpath `/grafana` |
| `values-alloy.yaml` | Alloy | OTLP + Faro receivers, exporters a Tempo/Loki/Prometheus |
| `values-tempo.yaml` | Tempo | Storage local, metrics_generator con 3 processors |
| `values-loki.yaml` | Loki | SingleBinary mode, filesystem storage, sin cache |
| `values-prometheus.yaml` | Prometheus | Remote write receiver, sin alertmanager/exporters |
| `k6-example-configmap.yaml` | k6 | Script de ejemplo para load testing |
| `k6-example-testrun.yaml` | k6 | CRD TestRun de ejemplo |

## Helm Repos

```bash
helm repo add grafana https://grafana.github.io/helm-charts
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```

## Comandos de Gestión

```bash
# Ver estado de todos los pods
kubectl get pods -n monitoring

# Upgrade de un componente (ejemplo: Tempo)
helm upgrade tempo grafana/tempo -n monitoring -f values-tempo.yaml

# Ver logs de Alloy
kubectl logs -n monitoring -l app.kubernetes.io/name=alloy -c alloy --tail=20

# Reiniciar un componente
kubectl rollout restart deployment alloy -n monitoring
kubectl delete pod tempo-0 -n monitoring  # StatefulSet

# Ver métricas generadas por Tempo
kubectl exec -n monitoring tempo-0 -- wget -qO- "http://localhost:3200/metrics" | grep metrics_generator

# Verificar traces en Tempo
kubectl exec -n monitoring tempo-0 -- wget -qO- "http://localhost:3200/api/search?limit=5"
```

## Istio VirtualServices

| VirtualService | Host | Path | Destino |
|---|---|---|---|
| `grafana-virtualservice` | services.kappali.com | `/grafana` | grafana.monitoring:80 |
| `alloy-faro-virtualservice` | services.kappali.com | `/alloy/collect` | alloy.monitoring:12347 |

## Recursos del Stack

| Componente | CPU req | RAM req | RAM limit | Storage |
|---|---|---|---|---|
| Alloy | 200m | 256 Mi | 512 Mi | - |
| Tempo | 200m | 512 Mi | 1 Gi | 30 Gi |
| Loki | 200m | 512 Mi | 1.5 Gi | 50 Gi |
| Prometheus | 100m | 256 Mi | 512 Mi | 10 Gi |
| Grafana | 100m | 256 Mi | 512 Mi | 5 Gi |
| **Total** | **800m** | **1.8 Gi** | **4 Gi** | **95 Gi** |

## Historial

- **2026-07-24**: Instalación inicial del stack completo
- **Motivo**: Grafana Cloud free tier insuficiente (10 dashboards, 14 días retención, 50 GB/mes)
- **Hallazgo crítico**: El OTel Collector anterior tenía un memory leak de 17.5 Gi (54% de RAM del servidor)
- **Resultado**: RAM del servidor bajó de 93% a 45% después de eliminar OTel Collector + optimizar infra
