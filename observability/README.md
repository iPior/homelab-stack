# Observability

See [planned work](TODO.md) for the remaining observability tasks.

Grafana comes with Prometheus configured as its default data source. Prometheus
collects metrics from itself, Grafana, Node Exporter, and cAdvisor. Node Exporter
reports host CPU, memory, filesystem, and network metrics. cAdvisor reports
resource usage for individual Docker containers.

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

## Homelab Overview dashboard

Grafana loads **Homelab Overview** under **Dashboards > Homelab**. The **Host**
selector filters exporter status, host metrics, and container usage. Hosts
with only Node Exporter still show host metrics. Container rankings show the
top ten containers across the selected hosts, using short Docker IDs when
cAdvisor does not report names.

On an existing installation, apply the dashboard mounts from this directory:

```bash
sudo docker compose up -d grafana
```

Edit `homelab-overview.json` to change the provisioned dashboard.

## cAdvisor on the media server and OptiPlex

From this directory on the media server, start cAdvisor and recreate Prometheus
to load the scrape jobs.

```bash
sudo docker compose up -d cadvisor
sudo docker compose up -d --force-recreate prometheus
```

Copy `docker-compose.optiplex.yml` to the OptiPlex or use it from a checkout of
this repository there. From the directory containing the file, run:

```bash
sudo docker compose -f docker-compose.optiplex.yml up -d
```

The OptiPlex publishes cAdvisor on its Tailscale address `100.75.212.97:9280`.
The media server's cAdvisor endpoint is on the Compose network. Both cAdvisor
containers run privileged with access to host files and the Docker socket.

In Grafana **Explore**, run `up{job="cadvisor"}` and look for `1` for both
`media-server` and `optiplex`. In **Homelab Overview**, check that each host
shows entries in the container memory ranking when selected individually.
