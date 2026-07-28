# Observabilidad — SigNoz self-hosted

## Arquitectura

```
                        ┌──────────────────────────────┐
                        │   SigNoz  (192.168.0.5)      │
                        │   Docker Compose · v0.134.0  │
                        │   UI 8080 · OTLP 4317/4318   │
                        └───────▲──────────────▲───────┘
                                │ gRPC 4317    │ HTTP 4318
                                │              │
        ┌───────────────────────┴───┐      ┌───┴─────────────────────────┐
        │  signoz-k8s-infra-        │      │  Istio Gateway              │
        │  otel-agent (DaemonSet)   │      │  signoz.kappali.com         │
        │  ns: signoz               │      │  ServiceEntry signoz.external│
        └───────────▲───────────────┘      └───▲─────────────────────────┘
                    │ OTLP gRPC                │ OTLP HTTP + CORS
                    │                          │
        ┌───────────┴───────────┐   ┌──────────┴──────────────┐
        │  ~72 microservicios   │   │  Kappali.Frontend       │
        │  .NET (ms-*-rest,     │   │  (navegador del usuario)│
        │  ms-*-worker)         │   │                         │
        └───────────────────────┘   └─────────────────────────┘
```

Los dos caminos son distintos a proposito: los microservicios salen por el agente del cluster (red
interna, gRPC), y el navegador sale por el ingress publico (HTTP, con CORS).

## Como se configura cada lado

**Microservicios.** Dos variables en cada `charts/ms-*/Staging.yaml`:

```yaml
LOGGER__OTELENDPOINT:      http://signoz-k8s-infra-otel-agent.signoz.svc.cluster.local:4317
OBSERVABILITY__SERVEROTEL: http://signoz-k8s-infra-otel-agent.signoz.svc.cluster.local:4317
```

El `service.name` sale de `{AppName}-{TypeEntryPoint}` (`ms-organization-rest`,
`ms-parking-worker`), y `deployment.environment` de `{{ .Values.environment }}`. Lo arma
`CodeDesignPlus.Net.Observability`, que propaga contexto en **W3C `traceparent`** — por eso la traza
del navegador se une con la del backend sin nada extra.

**Frontend.** No pasa por el agente: exporta directo a `https://signoz.kappali.com/v1/{traces,metrics,logs}`.
El endpoint esta en `runtimeConfig.public.otlp` de `nuxt.config.ts`, y el arranque en
`app/plugins/01.observability.client.ts`. Las reglas y las decisiones estan en
[`../../rules/13-observabilidad.md`](../../rules/13-observabilidad.md).

## Istio

[`istio-signoz.yaml`](istio-signoz.yaml) es una **copia byte a byte** del ServiceEntry, el
VirtualService y la AuthorizationPolicy que exponen SigNoz. Ahi esta el `corsPolicy` que permite que
el navegador exporte, y la `AuthorizationPolicy` que deniega cualquier metodo que no sea `POST` u
`OPTIONS` sobre las rutas OTLP.

**Ningun pipeline la aplica.** El original vive en `/opt/signoz-foundry/istio-signoz.yaml` en
`192.168.0.5` (`vm-monitoring`), y al cambiar algo hay que tocar los dos sitios. Para comprobar que
la copia sigue reflejando la realidad:

```bash
# Las tres lineas deben decir "unchanged". Si alguna dice "configured",
# el cluster y este fichero se han separado.
kubectl apply --dry-run=server -f helm-charts/monitoring/istio-signoz.yaml

# Y que la copia no se ha separado del original de la maquina:
ssh coded@192.168.0.5 cat /opt/signoz-foundry/istio-signoz.yaml \
  | diff - helm-charts/monitoring/istio-signoz.yaml
```

## Comprobaciones rapidas

```bash
# SigNoz vivo y con setup completo
curl -sS https://signoz.kappali.com/api/v1/version

# El preflight del navegador pasa (200 + cabecera allow-origin)
curl -sS -i -X OPTIONS https://signoz.kappali.com/v1/traces \
  -H "Origin: https://services.kappali.com" \
  -H "Access-Control-Request-Method: POST" | head -5

# Cualquier metodo que no sea POST/OPTIONS se deniega (403)
curl -sS -o /dev/null -w "%{http_code}\n" https://signoz.kappali.com/v1/traces

# El agente del cluster esta arriba y sabe a donde exportar
kubectl get pods -n signoz
kubectl get ds signoz-k8s-infra-otel-agent -n signoz \
  -o jsonpath='{.spec.template.spec.containers[0].env[?(@.name=="OTEL_EXPORTER_OTLP_ENDPOINT")].value}'
```

Un `POST` a `/v1/traces` que devuelve `200` **no** garantiza ingesta: hay que mirar el cuerpo.
`{"partialSuccess":{}}` significa que entro todo; si trae `rejectedSpans`, SigNoz lo recibio y lo
descarto.

## Ficheros historicos

Los `values-*.yaml` de este directorio (`grafana`, `alloy`, `tempo`, `loki`, `prometheus`) y los de
`k6-*` son de la stack de Grafana que se uso antes. **Ya no corre nada de eso**: el namespace
`monitoring` esta vacio, y `services.kappali.com/grafana` y `/alloy/collect` devuelven 404. Se
conservan solo como referencia de lo que hubo; no reflejan el estado del cluster.

Conviene saber que la instrumentacion Faro del frontend que ese montaje asumia **nunca llego a
existir**: habia configuracion en `nuxt.config.ts` pero ni dependencia ni plugin. El frontend estuvo
sin instrumentar hasta la llegada de SigNoz.

## Limitaciones conocidas

- **No hay session replay.** SigNoz no lo tiene y no hay equivalente.
- **No hay profiling continuo.** Se perdio al retirar Pyroscope.
- **Los stack traces del navegador llegan minificados.** Nuxt no emite sourcemaps de cliente en
  produccion, y SigNoz solo admite subirlos con su propio SDK web, no por OTLP generico.
- **SignalR no se instrumenta.** WebSockets con `skipNegotiation`, fuera del alcance de OTel.
