FROM otel/opentelemetry-collector-contrib:latest

# Copiar arquivo de configuração
COPY collector-config.yaml /conf/collector-config.yaml

# Expor portas necessárias
EXPOSE 4317 4318 9090

# Comando para executar o collector
CMD ["--config=/conf/collector-config.yaml"]
