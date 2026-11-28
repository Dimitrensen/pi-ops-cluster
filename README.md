# pi-ops-cluster

Home lab GitOps for a three-node k3s cluster: Hailo LLM chat, Argo CD, Prometheus, Grafana, Loki, and Alertmanager.

Host OS work (users, sudo, NTP, SSH keys, DietPi) lives in a **local** git repo, `lab-tools`, with no GitHub remote. This public repo is the Argo source of truth.

## Hardware

| Host | Role |
|---|---|
| `pi-cluster` | k3s control plane (Pironman 5). Grok, Slate MCP, and Ansible MCP run here. |
| `argo-pi` | k3s worker. Prometheus, Grafana, Loki, Alertmanager. |
| `ai-pi` | k3s worker + Hailo-10H AI HAT. `hailo-genai.service` and Open WebUI. |
| `pi-hole` | Pi Zero 2 W on argo-pi ethernet. Chrony NTP, node_exporter, blackbox. Not a k3s node. |

Remote access is Tailscale MagicDNS (`*.tail021710.ts.net`).

## Chat (Hailo)

- Model: `qwen2.5-coder:1.5b` on the Hailo HAT (`hailo-ollama` on `ai-pi:8000`)
- UI: Open WebUI NodePort **30081** — http://pi-cluster.tail021710.ts.net:30081
- Argo app: `hailo-qwen` (namespace `hailo-llm`). The Argo app name stays `hailo-qwen`; the chat objects stay `open-webui`.

## Observability

| Thing | URL / port |
|---|---|
| Grafana | http://pi-cluster.tail021710.ts.net:30300 |
| ai-pi temperature | Grafana → Lab → **ai-pi temperature** (alert at 70 °C for 2 minutes; Grafana UI only) |
| SSH logins | Grafana → Lab → **SSH logins**. Promtail ships `ssh.service` journal from the three k3s nodes. |
| Loki | Grafana Explore → Loki. Promtail on the three k3s nodes ships pod logs and SSH journal. |

Argo apps: `pi-cluster-root`, `hailo-qwen`, `observability`, `loki`, `promtail`.

## Ansible in this repo

Public `site.yml` only deploys the discreet **healthcheck** timer to the three k3s nodes (`roles/healthcheck` plus its jinja template).

```bash
ansible-playbook site.yml
```

## Sibling projects (this repo may refer to them)

- **lab-tools** (local only) — sudoers, pi-hole NTP, dedicated SSH key, `hailo-genai.service`, Ansible MCP.
- **slate-mcp** (https://github.com/Dimitrensen/slate-mcp) — shared blackboard MCP on `pi-cluster`. Independent repo. This cluster can use Slate; Slate must not depend on this repo.

Next after this baseline: MLflow, Kubeflow, an MLOps pipeline. Samsung laptop is available and not in GitOps yet.
