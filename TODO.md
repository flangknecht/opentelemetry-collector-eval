# Steps to take to evaluate OpenTelemetry tailbased sampling

- Create service creating spans for different systems
  - some attribute needs to be configurable to be able to differentiate between sampled and discarded spans
- Package service in container image
- Create k8s deployment descriptor
  - Generator deployment
    - Trace generator app
    - OTel Collector Agent sidecar
      - Use consistent hashing to route to individual backends by traceid
  - OTel Collector instances
    - Use tailbasedsamplingprocessor, sampling all traces with attribute `http.status_code` in [400;599]
  - Jaeger Collector + Jaeger Query + Jaeger UI (jaeger-all-in-one)

## Agent config

```yaml
receivers:
  otlp:
    protocols:
      grpc:
        endpoint: localhost:55680
processors:
exporters:
  logging:
  loadbalancing:
    protocol:
      otlp:
        # all options from the OTLP exporter are supported
        # except the endpoint
        timeout: 1s
    resolver:
      static:
        hostnames:
        - tailsamplingbackend-1:55680
        - tailsamplingbackend-2:55680
        - tailsamplingbackend-3:55680
service:
  pipelines:
    traces:
      receivers:
        - otlp
      processors: []
      exporters:
        - loadbalancing
```

## Tailsampling collector instance config


```yaml
receivers:
  otlp:
    protocols:
      grpc:
exporters:
  logging:
  jaeger:
    endpoint: jaeger:14250
    insecure: true
processors:
  tail_sampling:
    decision_wait: 10s
    policies:
      [
        {
          name: sample-http-errors, 
          type: numeric_attribute,
          numeric_attribute:
          {
            key: http.status_code,
            min_value: 400,
            max_value: 599
          }
        }
      ]
extensions:
  health_check:
  pprof:
    endpoint: :1888
  zpages:
    endpoint: :55679
service:
  extensions: [pprof, zpages, health_check]
  pipelines:
    traces:
      receivers: [otlp]
      processors: [tail_sampling]
      exporters: [logging, jaeger]
```
