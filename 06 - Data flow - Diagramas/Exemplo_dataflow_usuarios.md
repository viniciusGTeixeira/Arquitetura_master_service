# Exemplo de Data Flow - Cadastro de Usuário

---

## 1. Descrição
Este diagrama mostra o fluxo de dados no processo de cadastro de um novo usuário, do frontend ao banco de dados.

---

## 2. Diagrama Mermaid
````mermaid
%%{init: {'theme':'forest'}}%%
graph LR
    A[Usuário] --> B[Client Layer]
    B --> C[Master Layer]
    C --> D[Service Layer]
    D --> E[Banco de Dados]
    D --> F[Kafka]
    C --> G[Keycloak]
    G --> C
    C --> B
    B --> A
````

---

## 3. Explicação
1. O usuário preenche o formulário de cadastro no frontend.
2. Os dados são enviados para a Master Layer.
3. A Master Layer valida, autentica e encaminha para a Service Layer.
4. A Service Layer grava no banco e publica evento no Kafka.
5. O Keycloak é consultado para registro de identidade.
6. O fluxo retorna até o usuário com a confirmação. 