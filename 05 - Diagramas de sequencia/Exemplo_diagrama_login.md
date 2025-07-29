# Exemplo de Diagrama de Sequência - Login

---

## 1. Descrição
Este diagrama ilustra o fluxo de autenticação do usuário no sistema, desde o acesso inicial até a obtenção do token de sessão.

---

## 2. Diagrama Mermaid
````mermaid
%%{init: {'theme':'forest'}}%%
sequenceDiagram
    participant User as Usuário
    participant Client as Client Layer
    participant Master as Master Layer
    participant Keycloak as Keycloak

    User->>Client: Acessa página de login
    Client->>Master: Solicita chave pública
    Master-->>Client: Retorna chave RSA
    Client->>Client: Criptografa credenciais
    Client->>Master: Envia credenciais criptografadas
    Master->>Keycloak: Autentica usuário
    Keycloak-->>Master: Retorna JWT
    Master->>Client: Retorna token e sessão
    Client-->>User: Acesso liberado
````

---

## 3. Explicação
1. O usuário acessa a tela de login.
2. O frontend solicita a chave pública à Master Layer.
3. O usuário insere as credenciais, que são criptografadas e enviadas.
4. A Master Layer decripta, valida e autentica via Keycloak.
5. O token JWT é retornado e a sessão é iniciada. 