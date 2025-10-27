# 📊 Kubernetes - OpenTelemetry Collector para Monitoramento de Bancos de Dados

Este diretório contém a configuração Kubernetes para o OpenTelemetry Collector, projetado para monitorar múltiplos bancos de dados e exportar métricas para sistemas de observabilidade.

## 📁 Estrutura do Projeto

```
k8s/
├── kustomization.yaml         # Configuração principal do Kustomize
└── otel-collector/
    ├── collector-config.yaml  # Configuração do OpenTelemetry Collector
    ├── collector-deploy.yaml  # Deployment do Collector
    ├── collector-service.yaml # Service do Collector
    ├── secrets.yaml          # Secrets com credenciais
    └── README.md             # Documentação detalhada
```

## 🚀 Início Rápido

1. **Configure os secrets (não há arquivo .env no k8s):**
   ```bash
   cd k8s/otel-collector
   # Edite o arquivo secrets.yaml com suas credenciais
   ```

2. **Deploy usando Kustomize:**
   ```bash
   cd ..
   kubectl apply -k .
   ```

3. **Verifique o status:**
   ```bash
   kubectl get pods -n monitoring
   kubectl logs -f deployment/opentelemetrycollector -n monitoring
   ```

## 🗄️ Bancos de Dados Suportados

- **Redis**: Monitoramento de comandos, conexões, memória e chaves
- **PostgreSQL**: Monitoramento de backends, operações, tamanho do banco
- **MongoDB**: Monitoramento de hosts e operações
- **MySQL**: Monitoramento de buffer pool, comandos, conexões e threads
- **RabbitMQ**: Monitoramento de filas, mensagens, conexões e memória

## ⚙️ Configuração

### Secrets (secrets.yaml)

Configure as credenciais dos seus bancos de dados:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: otel-collector-secrets
  namespace: observability
type: Opaque
stringData:
  # Redis
  REDIS_ENDPOINT: "seu-redis.exemplo.com:6379"
  REDIS_PASSWORD: "sua-senha-redis"
  
  # PostgreSQL
  POSTGRES_ENDPOINT: "seu-postgres.exemplo.com:5432"
  POSTGRES_USERNAME: "usuario"
  POSTGRES_PASSWORD: "senha"
  
  # MongoDB
  MONGODB_ENDPOINT: "seu-mongo.exemplo.com:27017"
  MONGODB_USERNAME: "usuario"
  MONGODB_PASSWORD: "senha"
  
  # MySQL
  MYSQL_ENDPOINT: "seu-mysql.exemplo.com:3306"
  MYSQL_USERNAME: "usuario"
  MYSQL_PASSWORD: "senha"
  
  # RabbitMQ
  RABBITMQ_ENDPOINT: "seu-rabbitmq.exemplo.com:5672"
  RABBITMQ_USERNAME: "usuario"
  RABBITMQ_PASSWORD: "senha"
  
  # Observability
  TENANT_ID: "seu-tenant-id"
  API_TOKEN: "seu-token-de-api"
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

## 🌐 Serviços Expostos

O collector expõe os seguintes serviços:

| Porta | Protocolo | Descrição |
|-------|-----------|-----------|
| 4317 | gRPC | OTLP receiver |
| 4318 | HTTP | OTLP receiver |
| 9090 | HTTP | Métricas Prometheus |

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

## 🔧 Comandos Úteis

```bash
# Deploy com Kustomize (comando principal)
cd k8s && kubectl apply -k .

# Verificar status dos pods
kubectl get pods -n monitoring

# Ver logs do collector
kubectl logs -f deployment/opentelemetrycollector -n monitoring

# Verificar serviços
kubectl get svc -n monitoring

# Verificar secrets
kubectl get secrets -n monitoring

# Port-forward para testar localmente
kubectl port-forward svc/opentelemetrycollector 9090:9090 -n monitoring

# Testar métricas
curl http://localhost:9090/metrics

# Verificar configuração aplicada
kubectl describe deployment opentelemetrycollector -n monitoring

# Remover deployment
kubectl delete -k .
```

## 🔒 Segurança

- **Nunca commite** o arquivo `secrets.yaml` com credenciais reais
- Use namespaces dedicados para isolamento
- Configure NetworkPolicies se necessário
- Use RBAC para controlar acesso aos recursos
- Mantenha credenciais em sistemas de gerenciamento de segredos (ex: Vault)

## 🆘 Troubleshooting

### Problemas Comuns

1. **Pod não inicia**
   ```bash
   kubectl describe pod <pod-name> -n monitoring
   kubectl logs <pod-name> -n monitoring
   ```

2. **Não conecta nos bancos de dados**
   - Verifique se as credenciais estão corretas no secret
   - Confirme se os endpoints estão acessíveis do cluster
   - Teste conectividade de rede

3. **Métricas não aparecem**
   - Verifique se o pipeline está configurado corretamente
   - Confirme se o exporter está funcionando
   - Verifique logs do collector

4. **Erro de autenticação**
   - Verifique `TENANT_ID` e `API_TOKEN` no secret
   - Confirme se o endpoint do observability está correto

### Debugging

```bash
# Executar shell no pod para debug
kubectl exec -it deployment/opentelemetrycollector -n monitoring -- sh

# Verificar configuração dentro do pod
kubectl exec deployment/opentelemetrycollector -n monitoring -- cat /conf/collector-config.yaml

# Verificar variáveis de ambiente
kubectl exec deployment/opentelemetrycollector -n monitoring -- env | grep -E "(REDIS|POSTGRES|MONGODB|MYSQL|TENANT|API)"
```

## 📈 Escalabilidade

Para ambientes de alta carga:

1. **Aumentar replicas:**
   ```yaml
   # Em collector-deploy.yaml
   spec:
     replicas: 3
   ```

2. **Configurar recursos:**
   ```yaml
   # Em collector-deploy.yaml
   resources:
     requests:
       memory: "512Mi"
       cpu: "250m"
     limits:
       memory: "1Gi"
       cpu: "500m"
   ```

3. **Usar HPA (Horizontal Pod Autoscaler):**
   ```yaml
   apiVersion: autoscaling/v2
   kind: HorizontalPodAutoscaler
   metadata:
     name: otel-collector-hpa
     namespace: monitoring
   spec:
     scaleTargetRef:
       apiVersion: apps/v1
       kind: Deployment
       name: opentelemetrycollector
     minReplicas: 2
     maxReplicas: 10
     metrics:
     - type: Resource
       resource:
         name: cpu
         target:
           type: Utilization
           averageUtilization: 70
   ```

## 📚 Documentação Adicional

Para informações mais detalhadas sobre configuração e uso, consulte o [README.md](./otel-collector/README.md) na pasta `otel-collector`.

## 🆘 Suporte

Se você encontrar problemas ou tiver dúvidas:

1. Verifique os logs: `kubectl logs -f deployment/opentelemetrycollector -n monitoring`
2. Confirme se todos os secrets estão configurados corretamente
3. Teste a conectividade com os bancos de dados
4. Verifique se o endpoint do observability está acessível
5. Consulte a documentação do OpenTelemetry Collector
