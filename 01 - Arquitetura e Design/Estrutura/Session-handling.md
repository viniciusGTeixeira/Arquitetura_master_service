# Session Handling

### Versão: 1.0
### Data: 07-2025

## Objetivo

Este documento detalha o ciclo de vida das sessões: criação, validação e destruição, descrevendo os fluxos, integrações e mecanismos de segurança adotados.

---

## Armazenamento e Gerenciamento de Sessão no Banco de Dados

A Master Layer persiste e gerencia o ciclo de vida das sessões na tabela `sessions` do banco de dados do tenant. Isso garante rastreabilidade, controle centralizado e permite auditoria detalhada de todas as sessões ativas e encerradas.

### Modelo da Tabela `sessions`

| Campo           | Tipo           | Descrição                                 |
|-----------------|----------------|-------------------------------------------|
| id              | UUID           | Identificador único da sessão             |
| user_id         | UUID           | ID do usuário associado                   |
| tenant_id       | UUID           | ID do tenant                              |
| access_token    | VARCHAR        | Token JWT de acesso                       |
| refresh_token   | VARCHAR        | Token de refresh                          |
| context         | JSONB          | Contexto de permissões, roles, etc        |
| created_at      | TIMESTAMP      | Data/hora de criação da sessão            |
| expires_at      | TIMESTAMP      | Data/hora de expiração da sessão          |
| revoked_at      | TIMESTAMP      | Data/hora de revogação (se aplicável)     |
| ip_address      | VARCHAR        | IP de origem da sessão                    |
| user_agent      | VARCHAR        | User agent do cliente                     |
| last_activity   | TIMESTAMP      | Última atividade registrada               |
| status          | VARCHAR        | Status da sessão (ativa, expirada, revogada) |

### Implementação com Laravel Eloquent

A tabela `sessions` de cada tenant deve ser gerenciada utilizando o Eloquent ORM, garantindo integração nativa com o laravel, facilidade de manutenção e uso de recursos avançados.

#### Exemplo de Model Eloquent
```php
namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

class Session extends Model
{
    use SoftDeletes;

    protected $table = 'sessions';
    protected $primaryKey = 'id';
    public $incrementing = false;
    protected $keyType = 'string'; // UUID

    protected $fillable = [
        'id',
        'user_id',
        'tenant_id',
        'access_token',
        'refresh_token',
        'context',
        'created_at',
        'expires_at',
        'revoked_at',
        'ip_address',
        'user_agent',
        'last_activity',
        'status',
    ];

    protected $casts = [
        'context' => 'array',
        'created_at' => 'datetime',
        'expires_at' => 'datetime',
        'revoked_at' => 'datetime',
        'last_activity' => 'datetime',
    ];

    // Relacionamento com User
    public function user()
    {
        return $this->belongsTo(User::class);
    }

    // Relacionamento com Tenant
    public function tenant()
    {
        return $this->belongsTo(Tenant::class);
    }
}
```

#### Criação de Sessão
```php
Session::create([
    'id' => (string) Str::uuid(),
    'user_id' => $userId,
    'tenant_id' => $tenantId,
    'access_token' => $accessToken,
    'refresh_token' => $refreshToken,
    'context' => $contextArray,
    'expires_at' => $expiresAt,
    'ip_address' => $ip,
    'user_agent' => $userAgent,
    'status' => 'ativa',
]);
```

#### Validação de Sessão
```php
$session = Session::where('access_token', $accessToken)
    ->where('status', 'ativa')
    ->where('expires_at', '>', now())
    ->first();
```

#### Revogação de Sessão
```php
$session->update([
    'status' => 'revogada',
    'revoked_at' => now(),
]);
```

#### Observações e Boas Práticas
- Utilize UUID como chave primária para maior segurança e escalabilidade.
- Implemente SoftDeletes se desejar manter histórico de sessões removidas.
- Use eventos do Eloquent (created, updated, deleted) para acionar logs ou auditoria.
- Considere criar mutators/accessors para manipulação de campos como context.
- Relacione a sessão com User e Tenant para facilitar queries e integrações.
- Utilize policies e gates do Laravel para controle de acesso às sessões.

---

## Ajustes nos Fluxos

### Criação de Sessão
Após autenticação e validação do token, a Master Layer insere um novo registro na tabela `sessions` com os dados do usuário, tenant, tokens, contexto e metadados (IP, user agent, timestamps).

### Validação de Sessão
A cada requisição, além de validar o token, a Master Layer consulta a tabela `sessions` para garantir que a sessão está ativa, não expirada e não revogada. O campo `last_activity` pode ser atualizado para rastrear o uso.

### Destruição de Sessão
No logout, expiração ou revogação, a sessão correspondente é atualizada na tabela para status 'revogada' ou 'expirada', registrando o timestamp do evento.

#### Destruição da Sessão no Navegador (Laravel)
Após a revogação no banco de dados, é fundamental destruir a sessão do usuário no backend e no navegador para garantir que todos os dados de autenticação sejam removidos:

```php
// Revoga a sessão no banco (conforme já documentado)
$session->update([
    'status' => 'revogada',
    'revoked_at' => now(),
]);

// Encerra a sessão do usuário no backend
Auth::logout();
Session::flush(); // Remove todos os dados da sessão PHP

// Redireciona ou retorna resposta de logout
return redirect('/login')->with('message', 'Sessão encerrada com sucesso.');
```

> Observação: O uso de Session::flush() e Auth::logout() garante que todos os dados de sessão armazenados no backend sejam destruídos, protegendo contra reutilização indevida.

---

## 4. Considerações de Segurança

- **Criptografia:**  
  Todos os tokens são assinados (JWT RS256) e transmitidos via TLS 1.3.
- **Armazenamento Seguro:**  
  Tokens nunca são armazenados em localStorage/cookies inseguros; apenas em memória.
- **Rotação de Chaves:**  
  Chaves de assinatura são rotacionadas periodicamente.
- **Auditoria:**  
  Todas as ações de sessão são logadas para rastreabilidade.
- **Proteção contra Ataques:**  
  Rate limiting, CORS, XSS, CSRF e validação de origem são aplicados.

---

## 5. Monitoramento e Auditoria

- **Logs de Sessão:**  
  Cada criação, validação e destruição de sessão é registrada.
- **Métricas:**  
  Taxa de criação, expiração e revogação de sessões são monitoradas.
- **Alertas:**  
  Tentativas de uso de tokens inválidos ou revogados geram alertas de segurança.

---

## 6. Resiliência e Recuperação

- **Fallback:**  
  Em caso de falha do Keycloak, a Master Layer pode operar em modo restrito, negando novas sessões mas mantendo as válidas até expiração.
- **Backup:**  
  Sessões críticas podem ser persistidas temporariamente para recuperação em caso de falha.

---

## Conclusão

A gestão de sessões é central para a segurança, rastreabilidade e experiência do usuário. O uso de Keycloak, tokens JWT, políticas de expiração e revogação, aliado a práticas de auditoria e monitoramento, garante um ciclo de vida de sessão robusto, seguro e alinhado com as melhores práticas de arquitetura moderna.
