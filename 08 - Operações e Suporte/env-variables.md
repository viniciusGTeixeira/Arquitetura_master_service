# Variáveis de Ambiente
## Documentação de Variáveis Essenciais

### Versão: 1.0
### Data: 07-2025

---

## 1. Introdução

Este documento lista e explica as variáveis de ambiente necessárias para o funcionamento seguro e correto dos serviços do sistema.

---

## 2. Variáveis Comuns
| Variável              | Descrição                        | Exemplo                      |
|----------------------|----------------------------------|------------------------------|
| NODE_ENV             | Ambiente de execução             | production, staging, dev     |
| PORT                 | Porta do serviço                 | 3000                         |
| LOG_LEVEL            | Nível de log                     | info, debug, warn, error     |
| TZ                   | Timezone                         | America/Sao_Paulo            |

---

## 3. Segurança e Autenticação
| Variável              | Descrição                        | Exemplo                      |
|----------------------|----------------------------------|------------------------------|
| JWT_SECRET           | Segredo para assinar tokens JWT   | supersecretkey               |
| KEYCLOAK_URL         | URL do servidor Keycloak          | https://auth.empresa.com     |
| KEYCLOAK_REALM       | Realm do Keycloak                 | master-realm                 |
| KEYCLOAK_CLIENT_ID   | Client ID do Keycloak             | master-service               |
| KEYCLOAK_SECRET      | Segredo do client Keycloak        | <segredo>                    |

---

## 4. Banco de Dados (RDS)
| Variável              | Descrição                        | Exemplo                      |
|----------------------|----------------------------------|------------------------------|
| DB_HOST              | Host do banco de dados            | db.empresa.com               |
| DB_PORT              | Porta do banco                    | 5432                         |
| DB_NAME              | Nome do banco                     | app_db                       |
| DB_USER              | Usuário do banco                  | app_user                     |
| DB_PASSWORD          | Senha do banco                    | <senha>                      |                    |

---

## 5. Mensageria e Integrações
| Variável              | Descrição                        | Exemplo                      |
|----------------------|----------------------------------|------------------------------|
| KAFKA_BROKER_1       | Endereço do broker Kafka          | kafka1.empresa.com:9092      |
| KAFKA_BROKER_2       | Endereço do broker Kafka          | kafka2.empresa.com:9092      |
| API_THIRD_PARTY_URL  | URL de API externa                | https://api.externa.com      |

---

## 6. Boas Práticas
- Nunca versionar arquivos .env com segredos
- Usar variáveis diferentes por ambiente
- Rotacionar segredos periodicamente
- Validar variáveis obrigatórias no startup
