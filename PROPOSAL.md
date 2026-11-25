# Erdoel (Oil) - Cloud Computing Group 5
![Banner](imgs/banner.png)

**Members:** Julian Wieser, Jakob Guggenbichler, Jakob Defranceschi, Jannik ?

## Overview

We base our project on the [Pterodactyl Panel](https://github.com/pterodactyl/panel) and
[Pterodactyl Wings](https://github.com/pterodactyl/wings).

Pterodactyl is an open-source game server management platform. The **Panel** (PHP/Laravel + React) is the web UI and
API, **Wings** (Go) is the daemon on each node that starts every game server as its own Docker container.
Setting it up by hand takes many manual steps (database, Redis, web server, TLS, node registration), and out of the
box it gives the host almost no insight into resource usage or security of the running game servers.

## Goal:
Deploy Pterodactyl to the cloud reproducibly with a single pipeline run, and make the whole system
observable and secure: metrics, logs and alerts for the hosts and for every game server, plus runtime threat detection.

## Scope

1. **Infrastructure as Code:** everything is defined in [OpenTofu](https://github.com/opentofu/opentofu):
   - **Hosts:** for **production** OpenTofu provisions the cloud VMs (Panel + Wings nodes, network, firewall) and
     installs Docker via cloud-init. The **dev** environment is an existing self-hosted Docker server.
   - **Services:** a reusable OpenTofu module deploys Panel, database, Wings, the monitoring stack and Falco as
     containers through the Docker provider (over SSH), so the same code deploys to dev and production.
   - **State:** remote OpenTofu state, one workspace per environment.
2. **CI/CD:** [GitHub Actions](https://docs.github.com/actions) validates every pull request (`tofu validate`/`plan`),
   deploys to dev through a self-hosted runner on the dev server, and after manual approval (GitHub Environments) to
   production.
3. **Observability:** [OpenTelemetry Collector](https://github.com/open-telemetry/opentelemetry-collector),
   [Prometheus](https://github.com/prometheus/prometheus) and [Grafana](https://github.com/grafana/grafana) for
   metrics, dashboards and alerts, per host and per game server.
4. **Security:** [Falco](https://github.com/falcosecurity/falco) with custom rules detects suspicious activity on the
   hosts and inside game server containers and forwards alerts to Grafana.

### Stretch goals

- Horizontal scaling: changing `wings_node_count` in OpenTofu adds a new node that registers itself in the Panel via
  the Panel's Application API.
- OpenTelemetry tracing of the Panel (PHP auto-instrumentation).

## Out-of-Scope

- Changing Pterodactyl itself: we use it as it is and only extend it around the outside.
- Running Wings inside Kubernetes: Wings manages containers through the Docker daemon directly, so we deploy it on
  VMs with Docker.

## Demo (planned)

1. Run the pipeline on an empty cloud account -> working Panel + Wings node, create a Minecraft server from the Panel.
2. Grafana: overview dashboard and the per-server dashboard of the running game server.
3. Simulate load / memory exhaustion in a game server -> Prometheus alert fires.
4. `docker exec` a shell into a game server container -> Falco alert shows up in Grafana.
5. Open a pull request with a broken alert rule -> CI fails; fix it -> deploy to dev, approve -> production.

## Milestones

| #  | Due        | Milestone                                                                                    |
|----|------------|----------------------------------------------------------------------------------------------|
| M0 | 02.12.2026 | Proposal done, public GitHub repo, URL sent to all lecturers                                 |
| M1 | 09.12.2026 | OpenTofu deploys Panel and one Wings node to the dev server; a game server runs               |
| M2 | 16.12.2026 | GitHub Actions CI + two-stage CD (dev -> OpenTofu-provisioned cloud prod); Prometheus and Grafana running |
| M3 | 23.12.2026 | Per-game-server dashboards, alert rules, Falco with custom rules and alert forwarding        |
| M4 | 06.01.2027 | Final submission: README with tutorial, demo rehearsed |
| M5 | 13.01.2027 | Final presentation                                       |

## Responsibilities

| Member              | Main responsibility                                                 | Milestones |
|---------------------|---------------------------------------------------------------------|------------|
| ?       | Infrastructure as Code (OpenTofu, cloud-init), Pterodactyl deployment | M1         |
| Julian Wieser | CI/CD pipelines (GitHub Actions, dev/prod environments, validation) | M2         |
| ?             | Observability (OpenTelemetry Collector, Prometheus, Grafana)        | M2, M3     |
| Jakob Guggenbichler            | Security (Falco, custom rules, Falcosidekick alerting)              | M3         |
| All                 | README documentation, lessons learned, presentation and demo        | M4, M5     |
