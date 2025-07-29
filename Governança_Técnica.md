# Governança Técnica
## Padrões, Práticas e Responsabilidades

---

## 1. Padrões de Codificação
- Seguir guia de padrões de código do projeto (JS/TS, PHP, Vue)
- Uso obrigatório de ESLint, Prettier, PHPStan, PHP_CodeSniffer
- Commits padronizados (Conventional Commits)

---

## 2. Branches e Fluxo de Trabalho
- Branches principais: master, homolog, develop
- Branches de feature, bugfix, hotfix conforme guidelines
- Pull Requests obrigatórios para merge
- Revisão de código por pelo menos 1 membro

---

## 3. Práticas DevOps
- CI/CD automatizado (GitHub Actions)
- Testes automatizados em todos os merges
- Deploy automatizado para ambientes homolog e prod
- Monitoramento contínuo (Prometheus, Grafana)
- Backups automáticos e rotação de logs

---

## 4. Responsabilidades
- Dev: seguir padrões, escrever testes, revisar PRs
- DevOps: manter pipelines, monitoramento, infraestrutura
- SecOps: garantir segurança, revisar incidentes
- PO: priorizar backlog, aprovar releases

---

## 5. Revisão e Atualização
- Governança revisada a cada 6 meses ou a cada grande release

