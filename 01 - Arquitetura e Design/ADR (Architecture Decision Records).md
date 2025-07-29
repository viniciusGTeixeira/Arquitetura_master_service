# ADR - Architecture Decision Records
## Registro de Decisões Arquiteturais

### Versão: 1.0
### Data: 07-2025

## 1. Introdução
Este documento registra as principais decisões arquiteturais tomadas para os projetos, incluindo contexto, alternativas consideradas, decisão final e justificativa.

## 2. Decisões Arquiteturais

### ADR-001: Arquitetura em Camadas
- **Contexto**: Necessidade de separação de responsabilidades, escalabilidade e segurança.
- **Alternativas**: Monolito, microserviços,Serviços e Design em camadas(MRCS¹).
    [¹ - Model | Repository | Controller | Service]
- **Decisão**: Arquitetura em 3 camadas (Frontend - Backend) + camada de integrações externas.
- **Justificativa**: Facilita manutenção, escalabilidade e segurança centralizada.

### ADR-002: Autenticação via Keycloak
- **Contexto**: Requisito de autenticação centralizada, SSO e integração com múltiplos sistemas.
- **Alternativas**: Auth0, Keycloak, solução própria(Sanctum).
- **Decisão**: Keycloak via OpenID Connect.
- **Justificativa**: Open source, integração fácil, suporte a SSO e federação.

### ADR-003: Frontend em VueJS + Atomic Design
- **Contexto**: Necessidade de UI moderna, reusável e escalável.
- **Alternativas**: Angular, VueJS.
- **Decisão**: VueJS 3 com Atomic Design.
- **Justificativa**: Curva de aprendizado, produtividade, alinhamento com equipe.

### ADR-004: Orquestração Stateless na Master Layer
- **Contexto**: Escalabilidade e resiliência.
- **Alternativas**: Stateful, Stateless.
- **Decisão**: Stateless.
- **Justificativa**: Facilita auto scaling e failover.

### ADR-005: Event-driven e Mensageria (Kafka)
- **Contexto**: Integração assíncrona entre serviços e escalabilidade.
- **Alternativas**: REST síncrono,Kafka.
- **Decisão**: Kafka para mensageria e eventos.
- **Justificativa**: Alta performance, tolerância a falhas, escalabilidade.

### ADR-006: Repository Pattern e Service Layer no Service Layer
- **Contexto**: Manutenibilidade e clareza de domínio.
- **Alternativas**: Active Record, Repository Pattern, Service Layer.
- **Decisão**: Repository Pattern + Service Layer.
- **Justificativa**: Separação de domínio, testes, evolução.
