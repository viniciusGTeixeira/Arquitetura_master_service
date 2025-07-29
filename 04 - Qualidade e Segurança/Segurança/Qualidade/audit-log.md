# Audit Log
## Estratégia de Registro de Auditoria

### Versão: 1.0
### Data: 07-2025

---

## 1. Introdução

Definição de estratégia de registro de auditoria (audit log) do sistema, garantindo rastreabilidade, segurança e conformidade com requisitos legais e de negócio.

---

## 2. Eventos Registrados
- Autenticação e logout de usuários
- Alterações de permissões e roles
- Criação, edição e exclusão de dados sensíveis
- Acesso a dados pessoais e confidenciais
- Falhas de autenticação e autorização
- Operações administrativas e de configuração
- Tentativas de acesso negado

---

## 3. Formato do Log
```json
{
  "timestamp": "2024-12-31T10:00:00Z",
  "event_type": "UPDATE_USER",
  "user_id": "user-123",
  "tenant_id": "tenant-001",
  "request_id": "req-789",
  "correlation_id": "corr-012",
  "resource": "User",
  "resource_id": "user-123",
  "action": "update",
  "status": "success",
  "ip": "192.168.1.1",
  "details": {
    "fields_changed": ["email", "role"]
  }
}
```

---

## 4. Retenção e Acesso
- Retenção mínima: 5 anos
- Logs imutáveis (WORM)
- Acesso restrito a auditores e DPO
- Monitoramento de acessos ao log

---

## 5. Compliance
- LGPD: registro de acesso e alteração de dados pessoais
- SOX: rastreabilidade de operações críticas
