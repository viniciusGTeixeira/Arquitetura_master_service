# API Contracts Example
## Exemplo de Contrato de API RESTful

---

### Endpoint: Criar Usuário

- **URL**: `/api/v1/users`
- **Método**: POST
- **Auth**: Bearer Token (JWT)

#### Request
```json
{
  "name": "João Silva",
  "email": "joao@empresa.com",
  "password": "senhaSuperSecreta123",
  "tenant_id": "tenant-001"
}
```

#### Response 201
```json
{
  "status": "success",
  "message": "Usuário criado com sucesso",
  "data": {
    "id": "user-123",
    "name": "João Silva",
    "email": "joao@empresa.com",
    "tenant_id": "tenant-001"
  },
  "request_id": "req-789",
  "correlation_id": "corr-012",
  "timestamp": "2024-12-31T10:00:00Z"
}
```

#### Response 400
```json
{
  "status": "error",
  "error_code": "VALIDATION_ERROR",
  "message": "E-mail já cadastrado",
  "details": ["email"],
  "request_id": "req-789",
  "correlation_id": "corr-012"
}
```

#### Status Codes
- 201 Created
- 301 Permanent Redirect
- 400 Bad Request
- 401 Unauthorized
- 403 Forbidden
- 500 Internal Server Error