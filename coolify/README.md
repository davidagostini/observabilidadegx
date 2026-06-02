# Pasta Coolify

## Objetivo
Stack de observabilidade pronta para subir via Coolify, sem `ports` fixas no docker-compose.
O tráfego chega pelo domínio HTTPS configurado no painel do Coolify (proxy/SSL nativo da plataforma).

## Arquivos
- `docker-compose.coolify.yml`
- `app-remote.env.local-app.example`
- `app-remote.env.remote-app.example`
- `otel-collector.yaml`
- `tempo.yaml`
- `prometheus.yml`
- `grafana-datasources/datasources.yml`

## Deploy (passo a passo)
1. No Coolify, criar novo projeto/serviço e importar este repositório.
2. No serviço de `Docker Compose`, usar:
   - `Base Directory`: `/coolify`
   - `Docker Compose Location`: `/docker-compose.coolify.yml`
3. Configurar variáveis:
   - `GRAFANA_ADMIN_PASSWORD` com senha forte.
4. Habilitar domínio HTTPS em cada container que deseja publicar:
   - Grafana: `https://grafana.seudominio.com:3000`
   - Prometheus: `https://prometheus.seudominio.com:9090` (opcional)
   - Tempo: `https://tempo.seudominio.com:3200` (opcional)
   - OTEL Collector (somente se app for externa): `https://otel.seudominio.com:4318`
5. Subir.

Observacao: essa porta no campo de dominio e a porta interna do container para o proxy do Coolify. A URL publica continua sem porta, por exemplo `https://grafana.seudominio.com`.

## Opção A — app local (na mesma stack/rede)
- Use `app-remote.env.local-app.example`
- Variável principal:
  - `OTEL_EXPORTER_OTLP_ENDPOINT=http://otel-collector:4317`

## Opção B — app remota (fora da stack)
- Use `app-remote.env.remote-app.example`
- Configure domínio só com HTTPS, sem porta:
  - `OTEL_EXPORTER_OTLP_ENDPOINT=https://otel.seudominio.com`
  - `OTEL_EXPORTER_OTLP_PROTOCOL=http/protobuf`

Observação: para app remota via protocolo HTTP (recomendado com Coolify), mantenha o endpoint em HTTPS e sem porta.

### Observação de deploy no Coolify
- A configuração recomendada é exatamente `/coolify` + `/docker-compose.coolify.yml`.
- Se preferir base do repositório, use `Base Directory`: `/` e `Docker Compose Location`: `/docker-compose.coolify-root.yml`.
- Depois de atualizar o Git, clique em `Reload Compose File`, salve e force `Redeploy` para recriar os containers.
- No compose recarregado, os arquivos de configuração ficam embutidos no proprio `docker-compose.coolify.yml`.
- O compose do Coolify nao depende de bind mount nem de `configs` para Prometheus, Tempo, OTEL Collector ou datasource do Grafana.
