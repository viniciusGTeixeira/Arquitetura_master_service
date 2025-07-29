# SAD - Software Architecture Document
## Documento de Arquitetura de Software

### Versão: 1.0
### Data: 07-2025

---

## 1. Visão Geral

O sistema adota uma arquitetura de 3 camadas principais (Client, Master, Service) e uma camada de integrações externas (Third-Party), promovendo separação de responsabilidades, escalabilidade, segurança e manutenibilidade.

---

## 2. Estrutura Arquitetural

### 2.1 Diagrama de Alto Nível
````mermaid
%%{init: {'theme':'forest'}}%%
graph TD
    A[Client Layer]
    B[Master Layer]
    C[Service Layer]
    D[Third-Party Layer]
    A --> B
    B --> C
    C --> D
````

### 2.2 Componentes Principais
- **Client Layer**: Frontend VueJS, interface do usuário, criptografia RSA, gestão de sessão, atomic design.
- **Master Layer**: Orquestrador, autenticação Keycloak, roteamento, rate limiting, logging, auditoria, validação, gateway.
- **Service Layer**: Execução de regras de negócio, acesso a dados, cache, eventos, integração com mensageria e APIs externas.
- **Third-Party Layer**: Serviços externos, APIs, mensageria, sistemas legados.

---

## 3. Decisões Arquiteturais

### 3.1 Padrões e Princípios
- **Atomic Design** para frontend
- **SOLID** e **Service Layer** para backend
- **API-first** e contratos Swagger
- **Event-driven** para integração
- **Stateless** para Master Layer

### 3.2 Justificativas
- Separação clara de responsabilidades
- Escalabilidade horizontal
- Segurança centralizada
- Facilidade de manutenção e evolução

---

## 4. Detalhamento das Camadas

### 4.1 Client Layer
- VueJS 3, TypeScript, Pinia
- Criptografia RSA para credenciais
- JWT em memória, renovação automática
- Atomic Design para componentes

### 4.2 Master Layer
- PHP, Laravel
- Integração Keycloak (OpenID Connect)
- Rate limiting multi-nível
- Logging, auditoria, rastreamento
- Stateless, escalável

### 4.3 Service Layer
- PHP/Python, Kafka, Redis, PostgreSQL/MySQL
- Repository Pattern, Service Layer, SOLID
- Event sourcing e mensageria
- Cache multi-nível

### 4.4 Third-Party Layer
- Integração via REST/gRPC
- Circuit breaker, retry, fallback
- Adapters para sistemas legados

---

## 5. Diagramas Arquiteturais

### 5.1 Diagrama de Componentes
````mermaid
%%{init: {'theme':'forest'}}%%
graph TD
    subgraph Client
        A1[VueJS App]
        A2[Pinia Store]
        A3[Axios HTTP]
    end
    subgraph Master
        B1[API Gateway]
        B2[Keycloak Integration]
        B3[Rate Limiter]
        B4[Logger]
    end
    subgraph Service Layer
        C1[Business Logic]
        C2[Repository]
        C3[Service Class]
        C4[Event Publisher]
    end
    subgraph ThirdParty
        D1[External API]
        D2[Kafka]
        D3[Legacy Adapter - Legados]
    end
    A1 --> B1
    B1 --> B2
    B1 --> B3
    B1 --> B4
    B1 --> C1
    C1 --> C2
    C1 --> C3
    C3 --> D3
    C1 --> C4
    C4 --> D2
    C2 --> D1
    C3 --> D1
    C1 --> D3
````

### 5.2 Diagrama de Sequência (Login)
````mermaid
%%{init: {'theme':'forest'}}%%
sequenceDiagram
    participant Client
    participant Master
    participant Keycloak
    participant Service
    Client->>Master: Solicita chave pública
    Master-->>Client: Retorna chave RSA
    Client->>Master: Envia credenciais criptografadas
    Master->>Keycloak: Autentica usuário
    Keycloak-->>Master: Retorna JWT
    Master->>Client: Retorna token e sessão
    Client->>Master: Requisição autenticada
    Master->>Service: Encaminha requisição
    Service-->>Master: Resposta
    Master-->>Client: Resposta final
````

---

## 6. Qualidade e Segurança
- TLS 1.3, criptografia RSA/AES
- Rate limiting, CORS, XSS/CSRF/SQLi protection
- Logging, auditoria, rastreabilidade
- Backup, disaster recovery, alta disponibilidade