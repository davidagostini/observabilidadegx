# Pasta Local (Docker Compose)

## Objetivo
Stack de observabilidade para execução local com portas expostas diretamente.

## Arquivos
- `docker-compose.local-local-app.yml`
- `docker-compose.local-remote-app.yml`
- `app-remote.env.local-app.example`
- `app-remote.env.remote-app.example`
- `otel-collector.yaml`
- `tempo.yaml`
- `prometheus.yml`
- `grafana-datasources/datasources.yml`

## Opção 1 — App local (na mesma rede Docker)
1. Ajuste a aplicação para enviar OTEL para:
   - `http://otel-collector:4317`
2. Suba stack:
   - `docker compose -f docker-compose.local-local-app.yml up -d`

## Opção 2 — App remota (fora do Docker local)
1. Ajuste a aplicação para enviar OTEL para IP/DNS do host Docker:
   - `http://IP_DO_HOST_DOCKER:4317` (exemplo)
2. Suba stack:
   - `docker compose -f docker-compose.local-remote-app.yml up -d`

## Acesso
- Grafana: `http://localhost:3000` (ou porta/host que você definir no mapeamento)
- Prometheus: `http://localhost:9090`
- Tempo: `http://localhost:3200`

## Observação
Arquivos `app-remote.env.*.example` são exemplos; valores reais de senha/segredo devem ficar fora do repositório.
