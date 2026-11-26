# Observability backlog

- [ ] **Alerting.** Set up a Grafana contact point and verify delivery with a test notification. Add alerts for exporters unreachable for 5 minutes and filesystems below 10% free for 15 minutes.
- [ ] **PostgreSQL metrics.** Start with Immich's PostgreSQL instance. Collect connection counts, database size, and transaction activity using a PostgreSQL exporter with a dedicated monitoring user.
- [ ] **Application metrics.** Inventory which apps expose Prometheus metrics. Start with Home Assistant's sensor and entity metrics, then add endpoints for other services where useful.
- [ ] **Loki logs.** Add Loki and a log collector. Start with Docker logs from the media server and OptiPlex, set a retention period, and verify queries in Grafana Explore.
