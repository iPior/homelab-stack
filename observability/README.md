# Observability

Grafana comes with Prometheus configured as its default data source. Prometheus
collects metrics from itself, Grafana, and Node Exporter. Node Exporter reports
the media server's CPU, memory, filesystem, and network metrics. To monitor
other services, add their metrics endpoints or exporters to `prometheus.yml`
and recreate Prometheus.

From this directory, copy the environment template and set a unique
`GRAFANA_ADMIN_PASSWORD` in `.env`.

```bash
cp .env.example .env
chmod 600 .env
```

Then validate and start the stack.

```bash
docker compose config --quiet
docker compose up -d
docker compose ps
```

Open `http://localhost:3030` on the server and sign in as `admin` with the
password you set. For remote access, forward port 3030 over SSH or set
`GRAFANA_BIND_ADDRESS` to the server's LAN or Tailscale address. For example,
run `ssh -N -L 3030:127.0.0.1:3030 user@server` from your computer and open
`http://localhost:3030` there. Prometheus is reachable within the Compose network.

In Grafana, open **Explore**, select **Prometheus**, and run `up`. After the first
scrape, expect `1` for `prometheus`, `grafana`, and `node-exporter`. Run
`node_memory_MemAvailable_bytes{instance="media-server"}` to see available host
memory.

Node Exporter uses the Linux host network to read host interfaces and listens
on the Docker bridge address `172.17.0.1:9100`. Prometheus reaches it through
`host.docker.internal`. If your Docker daemon uses a custom host gateway, set
`NODE_EXPORTER_BIND_ADDRESS` to the matching IPv4 address in `.env`.

Grafana settings and Prometheus metrics persist in named Docker volumes. The
admin password in `.env` applies when Grafana initializes its database. Change
the password in Grafana after initialization. Prometheus retains up to 15 days
of metrics and also applies a 2 GB retention threshold.
