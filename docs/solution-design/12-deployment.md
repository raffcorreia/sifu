# 12 — Deploy e Ambiente

> **Camada:** Operations | **Depende de:** 03-architecture

## Estrutura de Repositório

```
sifu/
├── backend/               ← projeto Maven (Spring Boot)
├── frontend/              ← projeto Vite (React)
├── docker-compose.yml     ← ambiente local completo
├── docker-compose.test.yml ← ambiente de testes CI
└── docs/
```

## Docker Compose — Ambiente Local

```yaml
# docker-compose.yml
services:

  banco:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: sifu
      POSTGRES_USER: sifu
      POSTGRES_PASSWORD: ${DB_SENHA}
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD", "pg_isready", "-U", "sifu"]
      interval: 5s
      retries: 5

  backend:
    build: ./backend
    ports:
      - "8080:8080"
    environment:
      SPRING_DATASOURCE_URL: jdbc:postgresql://banco:5432/sifu
      SPRING_DATASOURCE_USERNAME: sifu
      SPRING_DATASOURCE_PASSWORD: ${DB_SENHA}
      JWT_SEGREDO: ${JWT_SEGREDO}
      CORS_ORIGENS_PERMITIDAS: http://localhost:5173
    depends_on:
      banco:
        condition: service_healthy

  frontend:
    build: ./frontend
    ports:
      - "5173:5173"
    environment:
      VITE_API_BASE_URL: http://localhost:8080
    depends_on:
      - backend

volumes:
  postgres_data:
```

## Variáveis de Ambiente

Arquivo `.env` na raiz (nunca commitado — ver `.gitignore`):

| Variável | Descrição | Exemplo |
|---|---|---|
| `DB_SENHA` | Senha do PostgreSQL | `s3nh4_s3gura` |
| `JWT_SEGREDO` | Segredo HMAC-SHA256 para JWT (mínimo 32 chars) | `segredo_muito_longo_aqui` |
| `CORS_ORIGENS_PERMITIDAS` | Origins aceitas pelo CORS | `http://localhost:5173` |
| `EMAIL_HOST` | Servidor SMTP para recuperação de senha | `smtp.example.com` |
| `EMAIL_PORTA` | Porta SMTP | `587` |
| `EMAIL_USUARIO` | Usuário SMTP | `noreply@sifu.gov.br` |
| `EMAIL_SENHA` | Senha SMTP | `senha_email` |
| `RATE_LIMIT_RPM` | Requisições por minuto por token de integração | `100` |

## Dockerfiles

### Backend (`backend/Dockerfile`)
```dockerfile
FROM maven:3.9-eclipse-temurin-21 AS build
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline
COPY src ./src
RUN mvn package -DskipTests

FROM eclipse-temurin:21-jre-alpine
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

### Frontend (`frontend/Dockerfile`)
```dockerfile
FROM node:22-alpine AS build
WORKDIR /app
COPY package*.json .
RUN npm ci
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=build /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
```

### `frontend/nginx.conf`

```nginx
server {
    listen 80;
    root /usr/share/nginx/html;
    index index.html;

    # SPA: redireciona todas as rotas para index.html
    location / {
        try_files $uri $uri/ /index.html;
    }

    # Cache de assets estáticos
    location ~* \.(js|css|png|jpg|svg|woff2)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }

    gzip on;
    gzip_types text/plain text/css application/javascript application/json;
}
```

## Como Iniciar o Ambiente Local

```bash
# 1. Clone o repositório
git clone <repo>
cd sifu

# 2. Configure o .env
cp .env.example .env
# edite .env com suas configurações

# 3. Suba o ambiente
docker compose up --build

# 4. Acesse
# Frontend: http://localhost:5173
# Backend API: http://localhost:8080/api/v1
# Swagger: http://localhost:8080/swagger-ui.html

# Login padrão (dados iniciais da migration V8):
# login: admin | senha: Admin@123
```

## CI/CD — GitHub Actions

```
.github/workflows/
├── ci.yml          ← roda em todo push e PR
└── deploy.yml      ← roda em push para main (Fase 2)
```

### ci.yml — jobs

```
backend-ci:
  1. Checkout
  2. Java 21 setup
  3. mvn verify (compila + testes + JaCoCo)
  4. Falha se cobertura < 80% / < 90% nos pacotes críticos

frontend-ci:
  1. Checkout
  2. Node 22 setup
  3. npm ci
  4. npm run lint
  5. npm test -- --coverage --watchAll=false
  6. Falha se cobertura < 70%
```

Merge em `main` bloqueado se qualquer job falhar.

## Migrações em Deploy

As migrações Flyway rodam automaticamente na inicialização do backend (`spring.flyway.enabled=true`). O backend aguarda o banco estar saudável (Docker healthcheck) antes de iniciar.

Em caso de falha na migração, o backend não inicializa — previne estado inconsistente.
