There now is a load balancing exporter that deterministically exports all spans belonging to the same trace ID to the same collector backend.

As pointed out in https://github.com/open-telemetry/opentelemetry-collector/issues/407#issuecomment-763729547, this can be used to set up a two-tiered collector infrastructure:

- Tier 1 (probably deployed as sidecar to the application or perhaps a DaemonSet) is configured to use consistent-hash load balancing across Trace IDs across all known available Tier 2 collector instances
- Tier 2 consists of collectors with tail-based sampling configured, since all spans belonging to a given trace ID will end up with one collector, no additional network chatter should be necessary
