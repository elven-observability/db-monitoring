# 📊 DB Monitoring - OpenTelemetry Collector

Solução completa de monitoramento de bancos de dados usando OpenTelemetry Collector, disponível tanto para **Kubernetes** quanto para **Docker**. Esta ferramenta coleta métricas de múltiplos bancos de dados e as exporta para sistemas de observabilidade como Prometheus/Mimir.

## 🎯 Características

- **Multi-banco**: Suporte para Redis, PostgreSQL, MongoDB e MySQL
- **Multi-plataforma**: Kubernetes e Docker
- **Flexível**: Configure apenas os bancos que você usa
- **Pronto para produção**: Configurações otimizadas e documentação completa
- **Observabilidade**: Integração com Prometheus/Mimir out-of-the-box

## 🗄️ Bancos de Dados Suportados

| Banco | Métricas Principais | Status |
|-------|-------------------|--------|
| **Redis** | Comandos, conexões, memória, chaves | ✅ |
| **PostgreSQL** | Backends, operações, tamanho do banco | ✅ |
| **MongoDB** | Hosts, operações, performance | ✅ |
| **MySQL** | Buffer pool, comandos, conexões, threads | ✅ |
| **RabbitMQ** | Filas, mensagens, conexões, memória | ✅ |

## 🚀 Início Rápido

### Opção 1: Docker (Desenvolvimento, teste e produção)

```bash
# Clone o repositório
git clone <seu-repositorio>
cd db-monitoring

# Configure as variáveis de ambiente
cd docker/otel-collector
cp env.example .env
# Edite o .env com suas credenciais

# Execute
cd ..
docker-compose up --build
```

### Opção 2: Kubernetes (Produção em clusters)

```bash
# Clone o repositório
git clone <seu-repositorio>
cd db-monitoring

# Configure os secrets (não há arquivo .env no k8s)
cd k8s/otel-collector
# Edite o secrets.yaml com suas credenciais

# Deploy usando Kustomize
cd ..
kubectl apply -k .
```

## 📁 Estrutura do Projeto

```
db-monitoring/
├── README.md                 # Este arquivo
├── docker/                   # Configuração Docker
│   ├── docker-compose.yml
│   ├── README.md
│   └── otel-collector/
│       ├── collector-config.yaml
│       ├── Dockerfile
│       ├── env.example
│       └── README.md
└── k8s/                      # Configuração Kubernetes
    ├── kustomization.yaml
    └── otel-collector/
        ├── collector-config.yaml
        ├── collector-deploy.yaml
        ├── collector-service.yaml
        ├── secrets.yaml
        └── README.md
```

## ⚙️ Configuração

### Docker: Variáveis de Ambiente (.env)

```bash
# Identificação do cliente/tenant
TENANT_ID=seu-tenant-id
API_TOKEN=seu-token-de-api

# Redis
REDIS_ENDPOINT=redis.exemplo.com:6379
REDIS_PASSWORD=sua-senha-redis

# PostgreSQL
POSTGRES_ENDPOINT=postgres.exemplo.com:5432
POSTGRES_USERNAME=usuario
POSTGRES_PASSWORD=senha

# MongoDB
MONGODB_ENDPOINT=mongo.exemplo.com:27017
MONGODB_USERNAME=usuario
MONGODB_PASSWORD=senha

# MySQL
MYSQL_ENDPOINT=mysql.exemplo.com:3306
MYSQL_USERNAME=usuario
MYSQL_PASSWORD=senha

# RabbitMQ
RABBITMQ_ENDPOINT=rabbitmq.exemplo.com:5672
RABBITMQ_USERNAME=usuario
RABBITMQ_PASSWORD=senha
```

### Kubernetes: Secrets (secrets.yaml)

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: otel-collector-secrets
  namespace: observability
type: Opaque
stringData:
  TENANT_ID: "seu-tenant-id"
  API_TOKEN: "seu-token-de-api"
  REDIS_ENDPOINT: "redis.exemplo.com:6379"
  REDIS_PASSWORD: "sua-senha-redis"
  POSTGRES_ENDPOINT: "postgres.exemplo.com:5432"
  POSTGRES_USERNAME: "usuario"
  POSTGRES_PASSWORD: "senha"
  MONGODB_ENDPOINT: "mongo.exemplo.com:27017"
  MONGODB_USERNAME: "usuario"
  MONGODB_PASSWORD: "senha"
  MYSQL_ENDPOINT: "mysql.exemplo.com:3306"
  MYSQL_USERNAME: "usuario"
  MYSQL_PASSWORD: "senha"
  RABBITMQ_ENDPOINT: "rabbitmq.exemplo.com:5672"
  RABBITMQ_USERNAME: "usuario"
  RABBITMQ_PASSWORD: "senha"
```

### 🔧 Personalização

**Não usa algum banco?** Simplesmente comente as seções correspondentes no `collector-config.yaml`:

```yaml
# Para desabilitar Redis, comente:
# receivers:
#   redis:
#     # ... configuração

# E também comente o pipeline:
# service:
#   pipelines:
#     metrics/redis:
#       # ... pipeline
```

**⚠️ IMPORTANTE: Personalize as Labels**

As labels na configuração são apenas exemplos. **Você DEVE alterar** as seguintes labels no arquivo `collector-config.yaml`:

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

## ✅ Checklist de Configuração

Antes de fazer o deploy, certifique-se de:

- [ ] **Credenciais configuradas**: Docker (.env) ou Kubernetes (secrets.yaml)
- [ ] **Labels personalizadas**: Alterar `instance` e `environment` no collector-config.yaml
- [ ] **Bancos não utilizados**: Comentados no collector-config.yaml
- [ ] **Endpoints corretos**: URLs e portas dos bancos de dados
- [ ] **Tokens válidos**: TENANT_ID e API_TOKEN configurados

## 📊 Métricas Coletadas

### Redis
- `redis.commands.processed` - Comandos processados
- `redis.connections.received` - Conexões recebidas
- `redis.memory.used` - Memória utilizada
- `redis.clients.connected` - Clientes conectados
- `redis.db.keys` - Chaves por banco

### PostgreSQL
- `postgresql.backends` - Backends ativos
- `postgresql.commits` - Commits realizados
- `postgresql.db_size` - Tamanho do banco
- `postgresql.operations` - Operações realizadas
- `postgresql.table.count` - Contagem de tabelas

### MongoDB
- Métricas de hosts e operações
- Performance e estatísticas do banco

### MySQL
- `mysql.buffer_pool.pages` - Páginas do buffer pool
- `mysql.commands` - Comandos executados
- `mysql.connections` - Conexões ativas
- `mysql.threads` - Threads do MySQL
- `mysql.operations` - Operações realizadas

### RabbitMQ
- `rabbitmq.queue.messages` - Mensagens nas filas
- `rabbitmq.queue.messages.ready` - Mensagens prontas para consumo
- `rabbitmq.queue.messages.unacknowledged` - Mensagens não confirmadas
- `rabbitmq.node.mem.used` - Memória utilizada do nó
- `rabbitmq.node.disk.free` - Espaço livre em disco
- `rabbitmq.connection.count` - Número de conexões ativas

## 🌐 Portas e Endpoints

| Porta | Protocolo | Descrição |
|-------|-----------|-----------|
| 4317 | gRPC | OTLP receiver |
| 4318 | HTTP | OTLP receiver |
| 9090 | HTTP | Métricas Prometheus |

## 🔒 Segurança

- **Nunca commite** arquivos `.env` ou `secrets.yaml` com credenciais reais
- Use tokens de API com permissões mínimas necessárias
- Configure TLS quando possível
- Mantenha credenciais em sistemas de gerenciamento de segredos

## 📚 Documentação Detalhada

- **[Docker Setup](./docker/README.md)** - Guia completo para Docker
- **[Kubernetes Setup](./k8s/otel-collector/README.md)** - Guia completo para Kubernetes

## 🆘 Troubleshooting

### Problemas Comuns

1. **Collector não conecta nos bancos**
   - Verifique se as credenciais estão corretas
   - Confirme se os endpoints estão acessíveis
   - Teste conectividade de rede

2. **Métricas não aparecem**
   - Verifique se o pipeline está configurado corretamente
   - Confirme se o exporter está funcionando
   - Verifique logs: `docker-compose logs` ou `kubectl logs`

3. **Erro de autenticação**
   - Verifique `TENANT_ID` e `API_TOKEN`
   - Confirme se o endpoint do observability está correto

### Comandos Úteis

```bash
# Docker - Ver logs
docker-compose logs -f otel-collector

# Docker - Testar métricas
curl http://localhost:9090/metrics

# Docker - Parar serviços
docker-compose down

# Kubernetes - Deploy com Kustomize
cd k8s && kubectl apply -k .

# Kubernetes - Ver logs
kubectl logs -f deployment/opentelemetrycollector -n monitoring

# Kubernetes - Verificar status
kubectl get pods -n monitoring

# Kubernetes - Port-forward para testar
kubectl port-forward svc/opentelemetrycollector 9090:9090 -n monitoring
```

## 🤝 Contribuição

1. Fork o projeto
2. Crie uma branch para sua feature (`git checkout -b feature/nova-feature`)
3. Commit suas mudanças (`git commit -am 'Adiciona nova feature'`)
4. Push para a branch (`git push origin feature/nova-feature`)
5. Abra um Pull Request

## 📄 Licença

Este projeto está sob a licença [MIT](LICENSE).

## 🆘 Suporte

Para suporte e dúvidas:
- Abra uma [issue](../../issues) no GitHub
- Consulte a documentação específica de cada plataforma
- Verifique os logs para diagnóstico de problemas

---

**Desenvolvido com ❤️ para facilitar o monitoramento de bancos de dados**
