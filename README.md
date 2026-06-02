# Pacote Completo Observabilidade (Coolify + Docker local)

Use este pacote para escolher entre:
- `coolify/` → para produção/orquestração no Coolify (sem portas host).
- `local/` → para testes/dev no Docker local (com portas host).

## Estrutura
- `coolify/`: compose sem `ports`, tudo pronto para domínio + SSL no painel.
- `local/`: compose com `ports`, com opção de app na mesma stack e app remota.
- `.gitignore`: protege `.env` e volume local.

## Fluxo recomendado de uso

### 1) Coolify
1. Crie repositório Git com esta pasta (`outputs/observabilidade-pacote-completo`).
2. No Coolify, adicione o app e selecione compose:
   - `coolify/docker-compose.coolify.yml`
3. Crie variável:
   - `GRAFANA_ADMIN_PASSWORD` (valor seguro)
4. Configure domínios no painel:
   - Grafana: `grafana.seudominio.com`
   - OTEL (opcional, só se app remota): `otel.seudominio.com`
5. Escolha `.env` de app no seu ambiente:
   - App interna à stack: `coolify/app-remote.env.local-app.example`
   - App remota: `coolify/app-remote.env.remote-app.example`
6. Deploy.

### 2) Docker local
1. Abra pasta `local/`.
2. Escolha:
   - `docker-compose.local-local-app.yml` (app local/rede do Docker)
   - `docker-compose.local-remote-app.yml` (app fora do Docker local)
3. Levante a stack:
   - `docker compose -f <arquivo> up -d`
4. Use as variáveis de exemplo equivalentes:
   - `local/app-remote.env.local-app.example`
   - `local/app-remote.env.remote-app.example`

## Segurança
- Não commitar arquivos `.env` reais.
- Preferir `OTEL_EXPORTER_OTLP_PROTOCOL=http/protobuf` + HTTPS em endpoint remoto público.
- Manter senhas e tokens apenas em secrets/variáveis do ambiente de execução.
