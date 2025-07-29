# Arquitetura Geral

## Visão

O sistema segue uma arquitetura de 3 camadas principais e 1 camada secundária:
- **Client Layer** – aplicações frontend ou consumidores da API.
- **Master Layer (Orchestrator)** – responsável por autenticar, validar, rotear.
- **Service Layer (Product Executor)** – executa a lógica de negócio, acessa bancos e repositórios.

- **Third_party Layer** - Serviços externos, apis externas, microserviços externos ou comunicação entre serviços a partir da camada Worker por conta do fluxo de produto

### Client Layer:
#### Front-end cliente ou sistema consumidor

<!-- - Neste caso entendemos o Client layer como o frontend(vueJS) ou sistema legado/monolito que consumirá nossa estrutura em algum nivel para suprir as necessidades de produto. -->

Esta é uma camada simples em termos gerais, não deve possuir nenhum grau de abstração ou acesso a informações restritas, não deve acessar diretamente nenhum Service layer, uma vez que ela não terá tratativas incisivas de segurança e deverá gerenciar somente dados recebidos sejam eles reativos ou não, esta etapa lidará com as respostas da camada Master e renderizará em tela, ou receberá dados que precisarão ser mantidos enquanto a sessão do usuario estiver ativas, neste caso esses dados só serão recebidos se a camada Client Layer contiver uma RSA_Public_Key válida. um exemplo disso é na etapa de acesso e autenticação, onde a chave publica (RSA_Public_Key) é utilizada em conjunto com a Lib Node-forge para encryptar as credenciais de acesso e enviar para a Master para seguir com o fluxo de login.

### Master Layer:
#### Camada intermediária – Master / Orquestrador

- A Master Layer é a camada central responsável por atuar como Master entre o Client Layer e os serviços internos (Service Layer). Seu papel é intermediar requisições, aplicar validações iniciais, gerenciar sessão e tokens, e garantir segurança e consistência dos acessos.

- Ela também é responsável por rastrear as requisições recebidas, aplicar lógicas de controle (rate-limiting, auditoria, autenticação e 2FA) e determinar qual serviço da Service Layer deverá ser invocado, com quais parâmetros, em qual path, seguindo que contrato e em que contexto.

- Toda entrada deve passar por validações explícitas (XSS, SQLi, tipo, range).

Responsabilidades principais:
- Validação de payloads recebidos (sanitização, estrutura, tipo de dado).
- Autenticação e autorização com base em tokens, chaves ou binários (JWT, RSA ou gRPC[protbuffer]).
- Orquestração de fluxos entre endpoints (ex: login, troca de senha, consumo de APIs internas).
- Gerenciamento de sessão e rastreamento (logs de atividade, tokens ativos).
- Proteção contra acessos indevidos (bloqueio de IP, revogação de sessão, antifraude inicial).

##### Ela atua com abstração da lógica de negócio, ou seja, ela não executa regras, apenas as encaminha.

**A Master deve ser:**

- Stateless, sempre que possível
- Escalável horizontalmente (com load balancer)
- Idealmente event-driven (ou pelo menos usar filas em segundo plano)

````mermaid
%%{init: {'theme':'forest'}}%%
stateDiagram-v2
    
    [*] --> CamadaServico

    state CamadaServico {
        direction LR
        state "Validação da Requisição" as Validacao
        state "Regras de Negócio" as RegrasNegocio
        state "Gerenciamento de Dados" as GerenciamentoDados
        
        Validacao --> RegrasNegocio
        RegrasNegocio --> GerenciamentoDados
    }

    state "Integrações" as Integracoes {
        direction LR
        BancoDados: Banco de Dados
        Cache: Memcached
        Fila: Fila Kafka
        ServicosExternos: APIs Externas
    }

    CamadaServico --> Integracoes
    
    state "Sistema de Logs" as SistemaLogs {
        direction TB
        Auditoria: Registro de Ações
        RastreioID: Rastreamento
        Metricas: Performance
    }

    CamadaServico --> SistemaLogs
    Integracoes --> SistemaLogs

    Integracoes --> [*]
````

**Exemplo de fluxo:**
No fluxo de login, após o Client Layer enviar as credenciais criptografadas, a Master Layer:

1. Decripta usando a chave RSA privada.
2. Valida o payload.
3. Verifica as credenciais via Keycloak (API).
4. recebe o token de acesso do keycloak (Access token JWT) 
5. Aplica camada anti time attack e envia a resposta padronizada.

````mermaid
%%{init: {'theme':'forest', 'themeVariables': {'sequenceNumberColor': '#FAFAF5'}}}%%
sequenceDiagram
    autonumber
    participant Client as Client Layer (Vue.js)
    participant Gateway as Master Layer (Orchestrator)
    participant Keycloak as Keycloak (Auth Server)

    Client->>Gateway: Requisição inicial (GET /public-key)
    Gateway-->>Client: Retorna RSA_Public_Key

    Client->>Client: Criptografa credenciais com RSA_Public_Key

    Client->>Gateway: Envia credenciais criptografadas
    activate Gateway

    Gateway->>Gateway: Decripta com RSA_Private_Key
    Gateway->>Gateway: Valida estrutura e campos do payload

    Gateway->>Keycloak: Requisição de autenticação (username/password)
    activate Keycloak

    Keycloak-->>Gateway: JWT Access Token (válido)
    deactivate Keycloak

    Gateway->>Gateway: Aplica delay anti time-attack
    Gateway-->>Client: Retorna token + dados da sessão
    deactivate Gateway

````

### Service Layer:
#### Camada de produtos ou regra de negócio - Worker
A proposta para a Service Layer nesta estrutura precisa refletir uma separação clara de responsabilidades, temos uma arquitetura e estrutura padrão adotada para nossos serviços baseada em **MACH e SOLID** e com base nisso o layer precisa:

- Executar a regra de negócios de forma isolada e coesa
- Receber e responder requisições da Master Layer (já autenticadas e autorizadas)
- Interagir com os repositórios de dados
- Interagir com os serviços externos (Third_Party)

**A Service Layer não deve aceitar chamadas diretas da Client Layer.**

- Ele precisa fornecer Responses com JSON padronizado **[status + mensagem + dados]** 
    - Nunca retornar senhas, hashes, erros de stack, lines ou qualquer estrutura baseada em codificação estática que favoreça falhas de segurança.
- Cada service deve ser testável de forma isolada.
- Cada serviço deve geranciar suas próprias filas Kafka.
- Cada serviço deve ter seu proprio sistema de logs que registre logging, audit, etc.


````mermaid
%%{init: {'theme':'forest'}}%%
stateDiagram-v2
    
    [*] --> CamadaServico

    state CamadaServico {
        direction LR
        state "Validação da Requisição" as Validacao
        state "Regras de Negócio" as RegrasNegocio
        state "Gerenciamento de Dados" as GerenciamentoDados
        
        Validacao --> RegrasNegocio
        RegrasNegocio --> GerenciamentoDados
    }

    state "Integrações" as Integracoes {
        direction LR
        BancoDados: Banco de Dados
        Fila: Fila Kafka
        ServicosExternos: APIs Externas
    }

    CamadaServico --> Integracoes
    
    state "Sistema de Logs" as SistemaLogs {
        direction TB
        Auditoria: Registro de Ações
        RastreioID: Rastreamento
        Metricas: Performance
    }

    CamadaServico --> SistemaLogs
    Integracoes --> SistemaLogs

    Integracoes --> [*]
````

### Third-Party Layer:
#### Camada de serviços e integrações externas

A camada Third-Party representa todos os serviços e integrações externas que nossa Service Layer precisa interagir. Esta camada é fundamental para:

- **Isolamento de Responsabilidades:**
  - Separar claramente integrações externas da lógica de negócio
  - Permitir substituição ou atualização de serviços externos sem impactar o core (camadas principais)
  - Facilitar o monitoramento e tratamento de falhas específicas de integrações

- **Tipos de Integrações possiveis:**
  - APIs REST de terceiros
  - Serviços de Mensageria externos
  - Sistemas legados da empresa
  - Microserviços de outros domínios
  - Serviços em nuvem (AWS)
  - Microserviços internos como a API v4 por exemplo

````mermaid
%%{init: {'theme':'forest'}}%%
sequenceDiagram
    autonumber
    participant SL as Service Layer
    participant TP as Third Party Layer
    participant EX as Serviços Externos

    SL->>TP: Requisição para serviço externo
    activate TP
    
    alt Circuito Fechado
        TP->>EX: Tentativa de requisição
        alt Sucesso
            EX-->>TP: Resposta OK
            TP-->>SL: Retorna dados processados
        else Falha Temporária
            EX-->>TP: Erro 5XX
            TP->>TP: Aplica política de retry
            TP->>EX: Nova tentativa
        else Falha Persistente
            TP->>TP: Abre circuit breaker
            TP-->>SL: Retorna fallback ou erro
        end
    else Circuito Aberto
        TP->>TP: Verifica tempo de recuperação
        TP-->>SL: Retorna resposta de fallback
    end
    
    deactivate TP
````


