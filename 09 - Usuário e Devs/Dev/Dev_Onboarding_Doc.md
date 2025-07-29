# Dev Onboarding Doc
## Guia de Onboarding para Desenvolvedores

### Versão: 1.0
### Data: 07-2025

---

## 1. Introdução

Bem-vindo ao time! Este guia tem como objetivo acelerar seu onboarding, apresentando o setup do ambiente, ferramentas, fluxo de trabalho e boas práticas do projeto.

---

## 2. Setup do Ambiente
- Clonar o repositório: `git clone <repo-url>`
- Instalar Node.js (>= 18), Podman, PHP (>= 8.3), e um editor recomendado (VSCode)
- Instalar dependências front: `npm install` ou `yarn`
- Instalar dependências back: `composer install` ou `brew` (em casos de macOS)
- Configurar variáveis de ambiente (.env.example)
- Subir containers: `podman-compose up -d`
- Rodar testes: `npm test`

---

## 3. Ferramentas Utilizadas
- Node.js, TypeScript, VueJS, Pinia
- PHP, Laravel
- Podman, Podman Compose Ou Docker, Docker-compose (Se for PJ e tiver essa liberdade)
- Kafka, PostgreSQL, MySQL
- ESLint, Prettier
- GitHub Actions (CI/CD)
- Jira

---

## 4. Fluxo de Trabalho
- Branches: `master`,`homolog`, `develop`, `feature/*`, `bugfix/*`
- Commits: seguir Conventional Commits
- Pull Requests: revisão obrigatória, checklist de PR
- Deploy: automatizado via CI/CD

---

## 5. Boas Práticas
- Escrever testes para novas features
- Documentar código e endpoints
- Seguir padrões de código (ESLint, Prettier)
- Revisar PRs de colegas

---

## 6. Contatos e Suporte
| Área         | Responsável       | Contato                |
|--------------|-------------------|------------------------|
| DevOps       | {        }        | devops@domain.com.br  |
| SecOps       | {        }        | secops@domain.com.br  |
| Produto      | {        }        | secops@domain.com.br  |
| Engenharia   | {        }        | secops@domain.com.br  |

---

## 7. Recursos Úteis
- Documentação do projeto (README.md)
- Wiki interna Ou Docs service (tst-uat.domain.com.br/domaindocs)
- Storybook para componentes (Figma)
