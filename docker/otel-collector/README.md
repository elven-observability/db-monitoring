# 📊 Configuração do OpenTelemetry Collector no Docker

Bem-vindo ao guia detalhado de configuração do OpenTelemetry Collector usando Docker. Neste guia, explicaremos como configurar o OpenTelemetry Collector usando Docker Compose e boas práticas para gerenciamento de credenciais.

## 📋 Pré-requisitos

Antes de começar, você deve ter o seguinte:

- **Docker** instalado e em execução
- **Docker Compose** instalado

## 🚦 Passo 1: Preparação dos Arquivos de Configuração

Certifique-se de que os seguintes arquivos estejam configurados corretamente em seu ambiente:

- **`docker-compose.yml`**
- **`otel-collector/collector-config.yaml`**
- **`otel-collector/Dockerfile`**
- **`otel-collector/env.example`**

## 📦 Passo 2: Configurar o Arquivo de Coleta - `collector-config.yaml`

O arquivo **`collector-config.yaml`** é fundamental para a configuração do OpenTelemetry Collector. Certifique-se de que ele esteja devidamente configurado. Este arquivo define como o OpenTelemetry Collector coleta, processa e exporta os dados de telemetria.

### ⚠️ IMPORTANTE: Personalize as Labels

**As labels na configuração são apenas exemplos!** Você DEVE alterar as seguintes labels:

```yaml
processors:
  resource/add_redis_labels:
    attributes:
      - action: insert
        key: instance
        value: "seu-redis-prod"  # ← ALTERE AQUI
      - action: insert
        key: environment
        value: "production"      # ← ALTERE AQUI
```

**Labels importantes para personalizar:**
- `instance`: Nome específico da sua instância do banco
- `environment`: Ambiente (dev, staging, production, etc.)
- `job`: Tipo do banco (redis, postgres, mongodb, mysql)

## 🐳 Passo 3: Configurar o Dockerfile - `Dockerfile`

O arquivo **`Dockerfile`** define como construir a imagem Docker do OpenTelemetry Collector. Ele usa a imagem oficial `otel/opentelemetry-collector-contrib:latest` e configura o arquivo de configuração.

## 🔧 Passo 4: Configurar as Variáveis de Ambiente - `env.example`

O arquivo **`env.example`** contém as variáveis de ambiente necessárias para a configuração do OpenTelemetry Collector. Copie este arquivo para `.env` e configure as credenciais corretas:

```bash
cp env.example .env
```

Exemplo de configuração no arquivo `.env`:

```bash
TENANT_ID=seu-tenant-id
API_TOKEN=seu-token-de-api
```

Lembre-se de que é uma boa prática manter essas credenciais em um cofre seguro e não commitar elas em repositórios públicos.

## 🚀 Passo 5: Executar com Docker Compose

Agora você pode executar o OpenTelemetry Collector usando Docker Compose:

```bash
# Construir e executar os serviços
docker-compose up --build

# Ou executar em background
docker-compose up -d --build
```

## 📊 Passo 6: Verificar o Status dos Serviços

Para verificar se os serviços estão funcionando corretamente:

```bash
# Ver logs do collector
docker-compose logs otel-collector

# Ver status dos containers
docker-compose ps

# Verificar métricas expostas
curl http://localhost:9090/metrics
```

## 🔄 Passo 7: Parar os Serviços

Para parar os serviços:

```bash
# Parar serviços
docker-compose down

# Parar e remover volumes
docker-compose down -v
```

## 🌐 Portas Expostas

O OpenTelemetry Collector expõe as seguintes portas:

- **4317**: gRPC OTLP receiver
- **4318**: HTTP OTLP receiver  
- **9090**: Prometheus metrics endpoint

## ✅ Conclusão

**📝** Com a configuração concluída, o OpenTelemetry Collector estará pronto para coletar e exportar telemetria usando Docker. Certifique-se de manter suas credenciais em segurança e não cometê-las em repositórios públicos. Se você encontrar algum problema ou tiver sugestões, não hesite em contribuir!

## 🔧 Comandos Úteis

```bash
# Reconstruir apenas o collector
docker-compose build otel-collector

# Ver logs em tempo real
docker-compose logs -f otel-collector

# Executar comandos dentro do container
docker-compose exec otel-collector sh

# Verificar configuração
docker-compose exec otel-collector cat /conf/collector-config.yaml
```
