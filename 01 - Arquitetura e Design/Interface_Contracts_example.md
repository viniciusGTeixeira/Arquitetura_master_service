# Interface Contracts Example
## Exemplo de Contrato de Interface (Frontend-Backend)

---

### Componente: UserCard.vue

#### Props
| Nome        | Tipo     | Obrigatório | Descrição                |
|-------------|----------|-------------|--------------------------|
| user        | Object   | Sim         | Dados do usuário         |
| showActions | Boolean  | Não         | Exibe botões de ação     |

#### Estrutura de Dados (user)
```json
{
  "id": "user-123",
  "name": "João Silva",
  "email": "joao@empresa.com",
  "role": "admin",
  "tenant_id": "tenant-001"
}
```

#### Eventos Emitidos
| Evento         | Payload           | Descrição                  |
|----------------|------------------|----------------------------|
| edit           | { id: string }   | Solicita edição do usuário |
| delete         | { id: string }   | Solicita exclusão          |

#### Exemplo de Uso
```vue
<UserCard :user="user" :showActions="true" @edit="onEdit" @delete="onDelete" />
```