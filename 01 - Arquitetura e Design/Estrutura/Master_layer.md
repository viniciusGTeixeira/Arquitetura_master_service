# Master Layer (Orchestrator)

## Visão Geral

A Master Layer é o componente central e crítico da arquitetura, atuando como orquestrador e gateway entre a camada cliente e os serviços. Esta camada é responsável por:
- Autenticação e autorização
- Validação e sanitização de dados
- Roteamento e orquestração de serviços
- Gestão de sessões e tokens
- Logging e auditoria
- Rate limiting e proteção

## Arquitetura Interna

````mermaid
%%{init: {'theme':'forest'}}%%
graph TB
    subgraph "Entrada"
        A[Load Balancer] --> B[Rate Limiter]
        B --> C[Request Handler]
    end

    subgraph "Segurança"
        C --> D[Security Layer]
        D --> E[Auth Service]
        D --> F[Token Manager]
        D --> G[Encryption Service]
        
        subgraph "Keycloak Integration"
            E --> E1[Keycloak HTTP Client]
            E1 --> E2[Keycloak API]
        end
    end

    subgraph "Processamento"
        D --> H[Request Validator]
        H --> I[Route Resolver]
        I --> J[Service Orchestrator]
    end

    subgraph "Monitoramento"
        K[Audit Logger]
        L[Performance Monitor]
        M[Error Handler]
    end

    J --> K
    J --> L
    J --> M
````

## Componentes Principais

### 1. Load Balancer e Rate Limiter

````mermaid
%%{init: {'theme':'forest'}}%%
graph LR
    subgraph "Load Balancing"
        A[Nginx Load Balancer] --> B[Master Node 1]
        A --> C[Master Node 2]
        A --> D[Master Node N]
    end

    subgraph "Rate Limiting"
        E[Global Rate Limit]
        F[Tenant Rate Limit]
        G[IP Rate Limit]
        H[User Rate Limit]
    end
````

#### Configurações de Rate Limiting
```yaml
rate_limits:
  global:
    requests_per_second: 10000
    burst: 1000
  per_tenant:
    requests_per_second: 1000
    burst: 100
  per_ip:
    requests_per_second: 100
    burst: 20
  per_user:
    requests_per_second: 50
    burst: 10
```

### 2. Camada de Segurança

#### Integração com Keycloak
A autenticação é gerenciada através do Keycloak, consumido via HTTP API. Esta integração fornece:

- **Gestão de Identidade**
  - Autenticação de usuários
  - Gerenciamento de roles e grupos
  - Single Sign-On (SSO)
  - Federação de identidade

````mermaid
%%{init: {'theme':'forest'}}%%
sequenceDiagram
    participant C as Client
    participant M as Master Layer
    participant KC as Keycloak API
    participant S as Service Layer

    C->>M: 1. Login Request
    
    rect rgb(200, 220, 200)
        Note over M,KC: Fluxo de Autenticação Keycloak
        M->>KC: 2. POST /auth/realms/{realm}/protocol/openid-connect/token
        KC-->>M: 3. Return Access + Refresh Tokens
    end

    M->>M: 4. Validate Token Structure
    M->>M: 5. Generate Session
    M-->>C: 6. Return Session Tokens

    rect rgb(200, 220, 200)
        Note over C,M: Requisições Subsequentes
        C->>M: 7. API Request + Bearer Token
        M->>KC: 8. Introspect Token
        KC-->>M: 9. Token Info
        M->>S: 10. Forward Request + Internal Context
    end
````

#### Configuração Keycloak
```yaml
keycloak:
  server_url: "https://auth.domain.com/auth"
  realm: "master-realm"
  client_id: "master-service"
  client_secret: "${KEYCLOAK_SECRET}"
  
  endpoints:
    token: "/protocol/openid-connect/token"
    userinfo: "/protocol/openid-connect/userinfo"
    logout: "/protocol/openid-connect/logout"
    introspect: "/protocol/openid-connect/token/introspect"
    
  settings:
    ssl_required: "external"
    verify_token_audience: true
    credentials:
      secret: "${KEYCLOAK_SECRET}"
    
  connection:
    pool_size: 20
    timeout: 5000
    keep_alive: true
```

#### Fluxo de Autenticação Detalhado

1. **Login Inicial**
   ```http
   POST /auth/realms/{realm}/protocol/openid-connect/token
   Content-Type: application/x-www-form-urlencoded

   grant_type=password&
   client_id=master-service&
   client_secret=${KEYCLOAK_SECRET}&
   username=user@domain.com&
   password=encrypted_password
   ```

2. **Resposta do Keycloak**
   ```json
   {
     "access_token": "eyJ...",
     "expires_in": 300,
     "refresh_expires_in": 1800,
     "refresh_token": "eyJ...",
     "token_type": "bearer",
     "session_state": "12345",
     "scope": "openid profile email"
   }
   ```

3. **Introspection de Token**
   ```http
   POST /auth/realms/{realm}/protocol/openid-connect/token/introspect
   Authorization: Bearer ${CLIENT_TOKEN}
   Content-Type: application/x-www-form-urlencoded

   token=${ACCESS_TOKEN}
   ```

4. **Resposta de Introspection**
   ```json
   {
     "active": true,
     "sub": "user-uuid",
     "email_verified": true,
     "roles": ["role1", "role2"],
     "name": "User Name",
     "preferred_username": "user@domain.com",
     "given_name": "User",
     "family_name": "Name",
     "email": "user@domain.com"
   }
   ```

#### Tratamento de Erros do Keycloak

````mermaid
%%{init: {'theme':'forest'}}%%
graph TB
    subgraph "Erros de Autenticação"
        A[Token Inválido] --> H[401 Unauthorized]
        B[Token Expirado] --> H
        C[Credenciais Inválidas] --> H
    end

    subgraph "Erros de Autorização"
        D[Permissão Insuficiente] --> I[403 Forbidden]
        E[Realm Inválido] --> I
    end

    subgraph "Erros de Sistema"
        F[Keycloak Indisponível] --> J[503 Service Unavailable]
        G[Timeout] --> J
    end
````

#### Estratégia de Cache e Performance

```typescript
interface KeycloakCache {
  // Cache de tokens introspectados
  tokenCache: Map<string, {
    tokenInfo: TokenIntrospection;
    expiresAt: number;
  }>;

  // Cache de configurações do realm
  realmCache: Map<string, {
    config: RealmConfiguration;
    fetchedAt: number;
  }>;

  // Cache de chaves públicas
  publicKeyCache: Map<string, {
    key: string;
    expiresAt: number;
  }>;
}
```

#### Monitoramento da Integração

Métricas específicas do Keycloak:
- Taxa de sucesso de autenticação
- Latência das chamadas à API
- Uso do cache (hit/miss ratio)
- Erros por tipo
- Tokens expirados/revogados
- Tempo médio de resposta

#### Gestão de Tokens
- **Access Token (JWT)**
  ```json
  {
    "typ": "JWT",
    "alg": "RS256",
    "kid": "master-key-1"
  }
  {
    "sub": "user-id",
    "tenant_id": "tenant-123",
    "roles": ["role1", "role2"],
    "permissions": ["perm1", "perm2"],
    "exp": 1735689600,
    "iat": 1735685000
  }
  ```

### 3. Validação e Sanitização

#### Pipeline de Validação
````mermaid
%%{init: {'theme':'forest'}}%%
graph LR
    A[Input] --> B[Schema Validation]
    B --> C[Type Check]
    C --> D[Business Rules]
    D --> E[Sanitization]
    E --> F[Valid Output]

    B -- Invalid --> X[Error Response]
    C -- Invalid --> X
    D -- Invalid --> X
    E -- Invalid --> X
````

#### Exemplo de Validação
```typescript
interface RequestValidator {
  validateSchema(payload: any): boolean;
  validateTypes(payload: any): boolean;
  validateBusinessRules(payload: any): boolean;
  sanitize(payload: any): any;
}

class PayloadValidator implements RequestValidator {
  // Implementação dos métodos
}
```

### 4. Orquestração de Serviços

#### Roteamento e Resolução
````mermaid
%%{init: {'theme':'forest'}}%%
graph TB
    A[Request] --> B{Route Resolver}
    B --> C[Service A]
    B --> D[Service B]
    B --> E[Service C]

    subgraph "Service Resolution"
        F[Parse Route]
        G[Load Config]
        H[Resolve Service]
        I[Build Context]
    end
````

#### Configuração de Rotas
```yaml
routes:
  - path: "/api/v1/users"
    service: "user-service"
    methods: ["GET", "POST"]
    auth_required: true
    validate_schema: "user.schema.json"
    rate_limit:
      requests_per_minute: 100
```

### 5. Logging e Auditoria

#### Estrutura de Logs
````mermaid
%%{init: {'theme':'forest'}}%%
graph TB
    subgraph "Log Types"
        A[Access Logs]
        B[Audit Logs]
        C[Error Logs]
        D[Performance Logs]
    end

    subgraph "Log Processing"
        E[Log Collector]
        F[Log Parser]
        G[Log Storage]
        H[Log Query]
    end

    A --> E
    B --> E
    C --> E
    D --> E
    E --> F
    F --> G
    G --> H
````

#### Formato de Log
```json
{
  "timestamp": "2024-03-20T10:00:00Z",
  "level": "INFO",
  "event_type": "REQUEST",
  "tenant_id": "tenant-123",
  "user_id": "user-456",
  "request_id": "req-789",
  "correlation_id": "corr-012",
  "service": "user-service",
  "path": "/api/v1/users",
  "method": "POST",
  "status_code": 200,
  "duration_ms": 150,
  "metadata": {
    "ip": "192.168.1.1",
    "user_agent": "Mozilla/5.0...",
    "resource_id": "user-789"
  }
}
```

### 6. Tratamento de Erros

#### Hierarquia de Erros
````mermaid
%%{init: {'theme':'forest'}}%%
graph TB
    A[Base Error] --> B[Validation Error]
    A --> C[Authentication Error]
    A --> D[Authorization Error]
    A --> E[Service Error]
    A --> F[System Error]

    B --> B1[Schema Error]
    B --> B2[Type Error]
    
    C --> C1[Invalid Token]
    C --> C2[Expired Token]
    
    D --> D1[Insufficient Permissions]
    D --> D2[Resource Access Denied]
    
    E --> E1[Service Unavailable]
    E --> E2[Timeout]
    
    F --> F1[Internal Error]
    F --> F2[Configuration Error]
````

#### Formato de Resposta de Erro
```json
{
  "status": "error",
  "code": "AUTH001",
  "message": "Token expirado",
  "details": {
    "token_expiry": "2024-03-20T09:00:00Z",
    "current_time": "2024-03-20T10:00:00Z"
  },
  "request_id": "req-789",
  "correlation_id": "corr-012"
}
```

## Configuração e Deploy

### Configuração de Ambiente
```yaml
master:
  nodes: 3
  port: 3000
  ssl:
    enabled: true
    cert_path: "/etc/ssl/master.crt"
    key_path: "/etc/ssl/master.key"
  
  security:
    rsa_key_size: 4096
    jwt_expiry: 3600
    refresh_token_expiry: 86400
    key_rotation_interval: 86400
    
  rate_limiting:
    enabled: true
    strategy: "sliding-window"
    
  logging:
    level: "info"
    format: "json"
    output: ["file", "stdout"]
    
  monitoring:
    metrics_enabled: true
    tracing_enabled: true
    health_check_interval: 30
```

### Estratégia de Deploy

````mermaid
%%{init: {'theme':'forest'}}%%
graph TB
    subgraph "Deploy Strategy"
        A[Build] --> B[Test]
        B --> C[Package]
        C --> D[Deploy]
        
        subgraph "Environments"
            E[Development]
            F[Staging]
            G[Production]
        end
        
        D --> E
        D --> F
        D --> G
    end
````

## Monitoramento e Métricas

### Métricas Principais
- Requests por segundo
- Latência média
- Taxa de erro
- Uso de memória
- CPU utilization
- Concurrent connections
- Token validation rate
- Cache hit/miss ratio

### Dashboard de Monitoramento
````mermaid
%%{init: {'theme':'forest'}}%%
graph TB
    subgraph "Monitoring Dashboard"
        A[System Metrics]
        B[Business Metrics]
        C[Security Metrics]
        D[Performance Metrics]
        
        subgraph "Alerts"
            E[High Error Rate]
            F[High Latency]
            G[Security Breach]
            H[Resource Exhaustion]
        end
    end
````

## Considerações de Segurança

### Proteções Implementadas
1. **Criptografia**
   - RSA para credenciais
   - TLS 1.3 para comunicação
   - Tokens JWT assinados

2. **Proteção contra Ataques**
   - Rate limiting
   - CORS configurado
   - XSS Protection
   - SQL Injection Protection
   - CSRF Tokens

3. **Auditoria**
   - Log de todas as ações
   - Rastreamento de sessões
   - Monitoramento de anomalias

### Políticas de Segurança
````mermaid
%%{init: {'theme':'forest'}}%%
graph TB
    subgraph "Security Policies"
        A[Access Control]
        B[Data Protection]
        C[Monitoring]
        D[Incident Response]
        
        A --> A1[Authentication]
        A --> A2[Authorization]
        
        B --> B1[Encryption]
        B --> B2[Data Handling]
        
        C --> C1[Logging]
        C --> C2[Alerting]
        
        D --> D1[Detection]
        D --> D2[Response]
    end
````

## Recuperação de Desastres

### Estratégias de Backup
- Backup automático de configurações
- Replicação de logs
- Snapshot de estado
- Backup de chaves e certificados

### Plano de Recuperação
1. Detecção de falha
2. Ativação de fallback
3. Restauração de backup
4. Validação de integridade
5. Retorno à operação normal

````mermaid
%%{init: {'theme':'forest'}}%%
sequenceDiagram
    participant S as System
    participant M as Monitor
    participant B as Backup
    participant R as Recovery

    S->>M: Health Check Fail
    M->>R: Trigger Recovery
    R->>B: Request Backup
    B-->>R: Provide Backup
    R->>S: Restore System
    S-->>M: Health Check OK
````

## Conclusão

A Master Layer é o componente mais crítico da arquitetura, responsável por garantir:
- Segurança e integridade
- Performance e escalabilidade
- Confiabilidade e disponibilidade
- Rastreabilidade e auditoria
- Gestão e monitoramento

Sua implementação robusta e bem planejada é fundamental para o sucesso do sistema como um todo.
