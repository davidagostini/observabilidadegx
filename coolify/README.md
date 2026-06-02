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
   - Grafana: `grafana.seudominio.com`
   - Prometheus: `prometheus.seudominio.com` (opcional)
   - Tempo: `tempo.seudominio.com` (opcional)
   - OTEL Collector (somente se app for externa): `otel.seudominio.com`
5. Subir.

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
- No compose recarregado, os mounts precisam aparecer como arquivo direto, por exemplo `./otelcol-config/config.yaml:/etc/otelcol-contrib/config.yaml:ro`.
