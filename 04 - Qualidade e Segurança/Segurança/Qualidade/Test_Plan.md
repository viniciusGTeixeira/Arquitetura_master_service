# Test Plan
## Plano de Testes do Sistema

### Versão: 1.0
### Data: 07-2025

---

## 1. Escopo

Este plano cobre os testes do sistema Master-Slave, incluindo frontend, backend, integrações e segurança.

---

## 2. Tipos de Teste
- Testes unitários
- Testes de integração
- Testes de contrato (API)
- Testes end-to-end (E2E)
- Testes de performance
- Testes de segurança (SAST, DAST, pentest)
- Testes de usabilidade

---

## 3. Critérios de Aceitação
- Todos os requisitos funcionais e não-funcionais cobertos
- Cobertura mínima de 80% para código crítico
- Zero blockers ou falhas críticas
- Performance: resposta ≤ 200ms para 95% das requisições
- Segurança: sem vulnerabilidades conhecidas

---

## 4. Responsabilidades
- QA: elaboração e execução dos testes
- Dev: criação de testes unitários e mocks
- PO: validação de critérios de aceitação
- Sec: execução de testes de segurança

---

## 5. Ferramentas
- Jest,PHPUnit (unitários)
- Cypress (E2E)
- Postman (API)
- SonarQube, Gallica - **IA de code review desenvolvida por Kemersson Teixeira** (código)
- OWASP ZAP, Snyk (segurança)
- Apche JMeter (performance)
