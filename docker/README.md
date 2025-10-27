# 📊 Docker - OpenTelemetry Collector para Monitoramento de Bancos de Dados

Este diretório contém a configuração Docker para o OpenTelemetry Collector, projetado para monitorar múltiplos bancos de dados e exportar métricas para sistemas de observabilidade como Prometheus/Mimir.

**Ideal para**: Desenvolvimento, testes, produção em EC2, VMs ou qualquer ambiente que suporte Docker.

## 📁 Estrutura do Projeto

```
docker/
├── docker-compose.yml          # Configuração principal do Docker Compose
└── otel-collector/
    ├── collector-config.yaml   # Configuração do OpenTelemetry Collector
    ├── Dockerfile             # Dockerfile para construir a imagem
    ├── env.example           # Exemplo de variáveis de ambiente
    └── README.md             # Documentação detalhada
```

## 🚀 Início Rápido

1. **Configure as variáveis de ambiente (.env):**
   ```bash
   cd otel-collector
   cp env.example .env
   # Edite o arquivo .env com suas credenciais
   ```

2. **Execute o serviço:**
   ```bash
   docker-compose up --build
   ```

3. **Verifique se está funcionando:**
   ```bash
   curl http://localhost:9090/metrics
   ```

> **💡 Dica**: Para produção, considere usar `docker-compose up -d --build` para executar em background.

## 🗄️ Bancos de Dados Suportados

- **Redis**: Monitoramento de comandos, conexões, memória e chaves
- **PostgreSQL**: Monitoramento de backends, operações, tamanho do banco
- **MongoDB**: Monitoramento de hosts e operações
- **MySQL**: Monitoramento de buffer pool, comandos, conexões e threads
- **RabbitMQ**: Monitoramento de filas, mensagens, conexões e memória

## 🔧 Configuração

### Variáveis de Ambiente Principais

- `TENANT_ID`: ID do tenant no sistema de observabilidade
- `API_TOKEN`: Token de autenticação para a API
- `REDIS_ENDPOINT`: Endpoint do Redis
- `POSTGRES_ENDPOINT`: Endpoint do PostgreSQL
- `MONGODB_ENDPOINT`: Endpoint do MongoDB
- `MYSQL_ENDPOINT`: Endpoint do MySQL
- `RABBITMQ_ENDPOINT`: Endpoint do RabbitMQ

### ⚠️ Personalização das Labels

**IMPORTANTE**: As labels na configuração são apenas exemplos. Você DEVE alterar as labels no arquivo `collector-config.yaml`:

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

  resource/add_postgres_labels:
    attributes:
      - action: insert
        key: instance
        value: "seu-postgres-prod"  # ← ALTERE AQUI
      - action: insert
        key: environment
        value: "production"          # ← ALTERE AQUI
```

**Labels importantes para personalizar:**
- `instance`: Nome específico da sua instância do banco
- `environment`: Ambiente (dev, staging, production, etc.)
- `job`: Tipo do banco (redis, postgres, mongodb, mysql)

### Portas Expostas

- **4317**: gRPC OTLP receiver
- **4318**: HTTP OTLP receiver
- **9090**: Prometheus metrics endpoint

## 📊 Métricas Coletadas

### Redis
- Comandos processados
- Conexões recebidas
- Chaves expiradas
- Memória utilizada
- Clientes conectados
- Chaves por banco

### PostgreSQL
- Backends ativos
- Blocos lidos
- Commits
- Contagem de bancos
- Tamanho do banco
- Operações
- Linhas processadas
- Contagem de tabelas

### MongoDB
- Hosts monitorados
- Operações do banco

### MySQL
- Páginas do buffer pool
- Comandos executados
- Conexões ativas
- Handlers
- Double writes
- Locks
- Operações
- Operações de página
- Row locks
- Threads
- Recursos temporários

### RabbitMQ
- Mensagens nas filas
- Mensagens prontas para consumo
- Mensagens não confirmadas
- Memória utilizada do nó
- Espaço livre em disco
- Número de conexões ativas

## 🔧 Comandos Úteis

```bash
# Executar em produção (background)
docker-compose up -d --build

# Ver logs em tempo real
docker-compose logs -f otel-collector

# Parar serviços
docker-compose down

# Reconstruir apenas o collector
docker-compose build otel-collector

# Executar comandos dentro do container
docker-compose exec otel-collector sh

# Verificar configuração
docker-compose exec otel-collector cat /conf/collector-config.yaml

# Verificar status dos containers
docker-compose ps
```

## 🔒 Segurança

- Mantenha suas credenciais em arquivos `.env` locais
- Não commite arquivos `.env` em repositórios públicos
- Use tokens de API com permissões mínimas necessárias

## 📚 Documentação Adicional

Para informações mais detalhadas sobre configuração e uso, consulte o [README.md](./otel-collector/README.md) na pasta `otel-collector`.

## 🆘 Suporte

Se você encontrar problemas ou tiver dúvidas:

1. Verifique os logs: `docker-compose logs otel-collector`
2. Confirme se todas as variáveis de ambiente estão configuradas
3. Teste a conectividade com os bancos de dados
4. Verifique se o endpoint do sistema de observabilidade está acessível
