# Metrics

## Network

irate(container_network_receive_bytes_total{namespace="otel"}[2m])
irate(container_network_transmit_bytes_total{namespace="otel"}[2m])

## CPU

irate(container_cpu_usage_seconds_total{namespace="otel",pod=~"tailsampling-collector-backend-.*"}[2m])*100

## Memory

container_memory_usage_bytes{namespace="otel",pod=~"tailsampling-collector-backend-.*"}/1000000
