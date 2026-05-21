# 11 — Observabilidade

> **Camada:** Operations | **Depende de:** 03-architecture

## Logging

### Backend (Spring Boot)

Formato estruturado JSON em produção (via Logback + logstash-logback-encoder):

```json
{
  "timestamp": "2025-05-20T14:30:00.000Z",
  "level": "INFO",
  "logger": "br.gov.sifu.notaempenho.NotaEmpenhoService",
  "mensagem": "NE emitida com sucesso",
  "numeroEmpenho": "2025MIN_ED000103",
  "ugId": 1,
  "usuarioLogin": "joao.silva",
  "duracao_ms": 45
}
```

Níveis de log por ambiente:

| Ambiente | Nível padrão | Nível fluxos críticos |
|---|---|---|
| Desenvolvimento | DEBUG | DEBUG |
| Produção | INFO | INFO |

Nunca logar: senhas, tokens em claro, dados pessoais sensíveis.

### Frontend (React)

Erros de API são capturados pelo interceptor Axios e enviados ao console em desenvolvimento. Em produção, erros não tratados são registrados no console sem expor detalhes internos ao usuário.

## Health Check

Spring Boot Actuator expõe:

```
GET /actuator/health
→ { "status": "UP", "components": { "db": { "status": "UP" } } }
```

Visível publicamente apenas o status agregado. Detalhes restritos a requisições internas.

## Métricas

Spring Boot Actuator + Micrometer expõem métricas compatíveis com Prometheus:

```
GET /actuator/metrics
GET /actuator/prometheus
```

Métricas monitoradas:
- `http.server.requests` — latência e contagem por endpoint e status HTTP
- `jvm.memory.used` — uso de memória JVM
- `hikaricp.connections.active` — pool de conexões do banco
- Métricas customizadas: empenhos emitidos por hora, NCs aprovadas, falhas de login

## Alertas Recomendados (Fase 2)

| Condição | Ação |
|---|---|
| `http.server.requests` p95 > 2s | Alerta de performance |
| Taxa de erro 5xx > 1% | Alerta de estabilidade |
| `hikaricp.connections.active` > 80% do pool | Alerta de capacidade |
| Falhas de login consecutivas > 10/min | Possível ataque de força bruta |
