# Pasta Local (Docker Compose)

## Objetivo
Stack completa de observabilidade para rodar localmente com Docker Compose, sem depender de GeneXus.

Ela sobe:
- Grafana
- Tempo
- Prometheus
- OpenTelemetry Collector

Qualquer aplicação compatível com OpenTelemetry pode enviar dados para este pacote: .NET, Java, Node.js, Python, GeneXus, containers Docker, serviços locais ou serviços em outra máquina.

## Arquivos principais
- `docker-compose.local-local-app.yml`
- `docker-compose.local-remote-app.yml`
- `app-remote.env.local-app.example`
- `app-remote.env.remote-app.example`
- `otelcol-config/config.yaml`
- `tempo-config/tempo.yaml`
- `prometheus-config/prometheus.yml`
- `grafana-datasources/datasources.yml`

## Quando usar cada compose

### Use `docker-compose.local-local-app.yml`
Use este arquivo quando a stack de observabilidade e a aplicação estiverem no mesmo ambiente local de desenvolvimento.

Cenários comuns:
- A aplicação também roda em container Docker na mesma rede.
- A aplicação roda na sua máquina local e envia para `localhost`.
- Você quer um ambiente simples de desenvolvimento/teste.
- Você não precisa guardar os dados do Grafana em uma pasta visível do projeto.

Neste compose, o Grafana usa um volume nomeado Docker:
- `grafana_data:/var/lib/grafana`

Isso é bom para desenvolvimento simples, porque o Docker gerencia o volume.

### Use `docker-compose.local-remote-app.yml`
Use este arquivo quando a stack roda localmente, mas quem envia telemetry está fora da stack.

Cenários comuns:
- Uma aplicação em outra máquina envia traces para seu Docker local.
- Uma aplicação em outro container/projeto Docker envia para o IP do host.
- Você quer expor o OpenTelemetry Collector pela rede local.
- Você quer os dados do Grafana em uma pasta do projeto para facilitar backup/remoção.

Neste compose, o Grafana usa uma pasta local:
- `./grafana_data:/var/lib/grafana`

Isso facilita ver, apagar ou copiar os dados locais do Grafana.

## Subir stack local simples
Na maioria dos testes locais, comece por este:

```powershell
git clone https://github.com/davidagostini/observabilidadegx.git
cd observabilidadegx\local
docker compose -f docker-compose.local-local-app.yml up -d
```

## Subir stack para receber dados de app externa/remota
Use este quando outra aplicação ou outra máquina for enviar dados para sua stack local:

```powershell
git clone https://github.com/davidagostini/observabilidadegx.git
cd observabilidadegx\local
docker compose -f docker-compose.local-remote-app.yml up -d
```

## Acessos locais
- Grafana: `http://localhost:3000`
- Prometheus: `http://localhost:9090`
- Tempo: `http://localhost:3200`
- OTEL Collector gRPC: `localhost:4317`
- OTEL Collector HTTP/protobuf: `http://localhost:4318`

Login padrão do Grafana:

```text
admin / admin
```

Para trocar a senha do Grafana:

```powershell
$env:GRAFANA_ADMIN_PASSWORD="sua-senha-forte"
docker compose -f docker-compose.local-local-app.yml up -d
```

## Como configurar aplicações

### Aplicação em container na mesma rede Docker
Use gRPC com o nome do serviço Docker:

```text
OTEL_EXPORTER_OTLP_ENDPOINT=http://otel-collector:4317
OTEL_EXPORTER_OTLP_PROTOCOL=grpc
OTEL_SERVICE_NAME=Minha-App
OTEL_TRACES_EXPORTER=otlp
OTEL_METRICS_EXPORTER=otlp
OTEL_LOGS_EXPORTER=otlp
```

### Aplicação rodando direto na sua máquina
Use HTTP/protobuf via `localhost`, que costuma ser mais simples:

```text
OTEL_EXPORTER_OTLP_ENDPOINT=http://localhost:4318
OTEL_EXPORTER_OTLP_PROTOCOL=http/protobuf
OTEL_SERVICE_NAME=Minha-App-Local
OTEL_TRACES_EXPORTER=otlp
OTEL_METRICS_EXPORTER=otlp
OTEL_LOGS_EXPORTER=otlp
```

### Aplicação em outra máquina na rede
Use o IP ou DNS da máquina que está rodando Docker:

```text
OTEL_EXPORTER_OTLP_ENDPOINT=http://IP_DO_HOST_DOCKER:4318
OTEL_EXPORTER_OTLP_PROTOCOL=http/protobuf
OTEL_SERVICE_NAME=Minha-App-Remota
OTEL_TRACES_EXPORTER=otlp
OTEL_METRICS_EXPORTER=otlp
OTEL_LOGS_EXPORTER=otlp
```

Se preferir gRPC:

```text
OTEL_EXPORTER_OTLP_ENDPOINT=http://IP_DO_HOST_DOCKER:4317
OTEL_EXPORTER_OTLP_PROTOCOL=grpc
```

## Portas que precisam estar liberadas

Para uso só na própria máquina:
- Não precisa abrir firewall externo.
- Use `localhost`.

Para receber dados de outra máquina:
- Liberar `4318/TCP` se usar HTTP/protobuf.
- Liberar `4317/TCP` se usar gRPC.
- Opcionalmente liberar `3000/TCP` se quiser acessar o Grafana de outra máquina.

Evite expor Prometheus (`9090`) e Tempo (`3200`) publicamente sem autenticação/rede privada.

## Validar se subiu

```powershell
docker compose -f docker-compose.local-local-app.yml ps
```

Health checks manuais:

```powershell
Invoke-WebRequest http://localhost:3000/api/health
Invoke-WebRequest http://localhost:9090/-/ready
Invoke-WebRequest http://localhost:3200/ready
```

O endpoint raiz do OTEL Collector pode retornar `404`; isso é normal. O importante é receber dados em:

```text
http://localhost:4318/v1/traces
http://localhost:4318/v1/metrics
http://localhost:4318/v1/logs
```

## Teste com trace fake
Se quiser gerar um trace de teste sem depender de aplicação real:

```powershell
docker run --rm ghcr.io/open-telemetry/opentelemetry-collector-contrib/telemetrygen:latest traces --otlp-endpoint host.docker.internal:4317 --service observabilidade-local-test --traces 3
```

Depois abra o Grafana:

```text
http://localhost:3000
```

Vá em:

```text
Explore > Tempo
```

Use a query:

```traceql
{ resource.service.name = "observabilidade-local-test" }
```

## Ver logs

```powershell
docker compose -f docker-compose.local-local-app.yml logs -f otel-collector
docker compose -f docker-compose.local-local-app.yml logs -f tempo
docker compose -f docker-compose.local-local-app.yml logs -f prometheus
docker compose -f docker-compose.local-local-app.yml logs -f grafana
```

## Parar

```powershell
docker compose -f docker-compose.local-local-app.yml down
```

Para apagar também os volumes:

```powershell
docker compose -f docker-compose.local-local-app.yml down -v
```

## Observações importantes
- Arquivos `app-remote.env.*.example` são exemplos; valores reais de senha/segredo devem ficar fora do repositório.
- Para filtro no Grafana/Tempo, sempre defina `OTEL_SERVICE_NAME`.
- Tempo armazena traces; métricas aparecem no Prometheus.
- Logs enviados ao OTEL Collector não são persistidos em Loki neste pacote.
- Se o serviço aparecer no Grafana mas sem conteúdo, aumente o período para `Last 1 hour` e filtre por `resource.service.name`.
