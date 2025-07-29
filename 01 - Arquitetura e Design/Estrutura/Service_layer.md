# Service Layer (Worker)

## Visão Geral

A Service Layer é responsável pela execução da lógica de negócio, processamento de dados e integração com repositórios. Esta camada atua como worker, recebendo requisições autenticadas e autorizadas da Master Layer, executando as regras de negócio e retornando respostas padronizadas.

## Arquitetura Interna

````mermaid
%%{init: {'theme':'forest'}}%%
graph TB
    subgraph "Service Layer"
        A[Request Handler] --> B[Validation Layer]
        B --> C[Business Logic]
        C --> D[Data Access Layer]
        
        subgraph "Business Logic"
            E[Domain Services]
            F[Use Cases]
            G[Business Rules]
        end
        
        subgraph "Data Access"
            H[Repositories]
            I[Data Mappers]
            J[Cache Layer]
        end
        
        subgraph "External Integration"
            K[Third Party Gateway]
            L[Message Queues]
            M[Event Publisher]
        end
    end

    subgraph "Infrastructure"
        N[Database]
        O[Cache Redis]
        P[Kafka Queues]
        Q[External APIs]
    end

    D --> H
    H --> N
    I --> O
    L --> P
    K --> Q
````

## Princípios Arquiteturais

### MACH (Microservices, API-first, Cloud-native, Headless)
- **Microservices**: Cada serviço é independente e focado em um domínio[contexto] específico
- **API-first**: APIs bem definidas como contratos principais
- **Cloud-native**: Projetado para ambientes distribuídos e escaláveis
- **Headless**: Desacoplado da camada de apresentação

### SOLID
- **Single Responsibility**: Cada classe/módulo tem uma única responsabilidade
- **Open/Closed**: Aberto para extensão, fechado para modificação
- **Liskov Substitution**: Subtipos devem ser substituíveis por seus tipos base
- **Interface Segregation**: Interfaces específicas são melhores que interfaces gerais
- **Dependency Inversion**: Depender de abstrações, não de concretizações

## Componentes Principais

### 1. Request Handler

````mermaid
%%{init: {'theme':'forest'}}%%
graph LR
    A[Master Layer Request] --> B[Request Handler]
    B --> C[Context Extraction]
    C --> D[Payload Validation]
    D --> E[Business Logic Router]
    E --> F[Response Builder]
    F --> G[Standardized Response]
````

#### Estrutura de Request
```typescript
interface ServiceRequest {
  correlation_id: string;
  request_id: string;
  tenant_id: string;
  user_id: string;
  timestamp: string;
  service: string;
  action: string;
  payload: any;
  context: {
    roles: string[];
    permissions: string[];
    session_id: string;
  };
}
```

#### Estrutura de Response
```typescript
interface ServiceResponse {
  status: 'success' | 'error';
  message: string;
  data?: any;
  error_code?: string;
  details?: any;
  request_id: string;
  correlation_id: string;
  timestamp: string;
  execution_time_ms: number;
}
```

### 2. Validation Layer

#### Pipeline de Validação
````mermaid
%%{init: {'theme':'forest'}}%%
graph LR
    A[Input] --> B[Schema Validation]
    B --> C[Business Rules Validation]
    C --> D[Permission Check]
    D --> E[Data Integrity Check]
    E --> F[Valid Input]

    B -- Invalid --> X[Validation Error]
    C -- Invalid --> X
    D -- Unauthorized --> Y[Authorization Error]
    E -- Invalid --> Z[Integrity Error]
````

#### Exemplo de Validação
```typescript
interface RequestValidator {
  validateSchema(payload: any): ValidationResult;
  validateBusinessRules(payload: any, context: RequestContext): ValidationResult;
  validatePermissions(action: string, context: RequestContext): AuthorizationResult;
  validateDataIntegrity(payload: any): IntegrityResult;
}

class UserServiceValidator implements RequestValidator {
  validateSchema(payload: any): ValidationResult {
    // Validação de schema usando Joi ou similar
    return {
      isValid: true,
      errors: []
    };
  }

  validateBusinessRules(payload: any, context: RequestContext): ValidationResult {
    // Regras específicas do domínio de usuários
    return {
      isValid: true,
      errors: []
    };
  }
}
```

### 3. Business Logic

#### Domain Services
````mermaid
%%{init: {'theme':'forest'}}%%
graph TB
    subgraph "Domain Services"
        A[User Service]
        B[Order Service]
        C[Payment Service]
        D[Notification Service]
        E[Audit Service]
    end

    subgraph "Shared Services"
        F[Encryption Service]
        G[Validation Service]
        H[Logging Service]
        I[Cache Service]
    end

    A --> F
    B --> G
    C --> H
    D --> I
````

#### Use Cases (Casos de Uso)
```typescript
interface UseCase<TInput, TOutput> {
  execute(input: TInput, context: RequestContext): Promise<TOutput>;
}

class CreateUserUseCase implements UseCase<CreateUserInput, UserOutput> {
  constructor(
    private userRepository: UserRepository,
    private encryptionService: EncryptionService,
    private auditService: AuditService
  ) {}

  async execute(input: CreateUserInput, context: RequestContext): Promise<UserOutput> {
    // 1. Validar dados de entrada
    this.validateInput(input);

    // 2. Verificar se usuário já existe
    const existingUser = await this.userRepository.findByEmail(input.email);
    if (existingUser) {
      throw new BusinessError('USER_ALREADY_EXISTS', 'Usuário já existe');
    }

    // 3. Criptografar senha
    const hashedPassword = await this.encryptionService.hash(input.password);

    // 4. Criar usuário
    const user = new User({
      ...input,
      password: hashedPassword,
      tenant_id: context.tenant_id,
      created_by: context.user_id
    });

    // 5. Salvar no repositório
    const savedUser = await this.userRepository.save(user);

    // 6. Registrar auditoria
    await this.auditService.log({
      action: 'USER_CREATED',
      entity_type: 'User',
      entity_id: savedUser.id,
      user_id: context.user_id,
      tenant_id: context.tenant_id
    });

    // 7. Retornar resultado
    return this.mapToOutput(savedUser);
  }
}
```

### 4. Data Access Layer

#### Repository Pattern
````mermaid
%%{init: {'theme':'forest'}}%%
graph TB
    subgraph "Repository Layer"
        A[IUserRepository] --> B[UserRepository]
        C[IOrderRepository] --> D[OrderRepository]
        E[IPaymentRepository] --> F[PaymentRepository]
    end

    subgraph "Data Mappers"
        G[User Entity] --> H[User Model]
        I[Order Entity] --> J[Order Model]
    end

    subgraph "Database"
        K[PostgreSQL]
        L[MySQL]
        M[Redis Cache]
    end

    B --> K
    D --> L
    F --> M
````

#### Interface de Repository
```typescript
interface Repository<T, ID> {
  findById(id: ID): Promise<T | null>;
  findAll(criteria?: SearchCriteria): Promise<T[]>;
  save(entity: T): Promise<T>;
  update(id: ID, entity: Partial<T>): Promise<T>;
  delete(id: ID): Promise<boolean>;
}

interface UserRepository extends Repository<User, string> {
  findByEmail(email: string): Promise<User | null>;
  findByTenant(tenantId: string): Promise<User[]>;
  findActiveUsers(): Promise<User[]>;
}
```

### 5. Cache Strategy

````mermaid
%%{init: {'theme':'forest'}}%%
graph LR
    A[Request] --> B{Cache Hit?}
    B -- Yes --> C[Return Cached Data]
    B -- No --> D[Fetch from Database]
    D --> E[Store in Cache]
    E --> F[Return Data]

    subgraph "Cache Layers"
        G[L1 - Memory Cache]
        H[L2 - Redis Cache]
        I[L3 - Database]
    end
````

#### Implementação de Cache
```typescript
interface CacheService {
  get<T>(key: string): Promise<T | null>;
  set<T>(key: string, value: T, ttl?: number): Promise<void>;
  delete(key: string): Promise<void>;
  invalidatePattern(pattern: string): Promise<void>;
}

class RedisCacheService implements CacheService {
  private readonly defaultTTL = 3600; // 1 hora

  async get<T>(key: string): Promise<T | null> {
    const value = await this.redis.get(key);
    return value ? JSON.parse(value) : null;
  }

  async set<T>(key: string, value: T, ttl = this.defaultTTL): Promise<void> {
    await this.redis.setex(key, ttl, JSON.stringify(value));
  }
}
```

### 6. Event System

#### Event-Driven Architecture
````mermaid
%%{init: {'theme':'forest'}}%%
graph TB
    subgraph "Event Flow"
        A[Business Logic] --> B[Event Publisher]
        B --> C[Kafka Topic]
        C --> D[Event Subscribers]
        D --> E[Event Handlers]
    end

    subgraph "Event Types"
        F[Domain Events]
        G[Integration Events]
        H[System Events]
    end
````

#### Estrutura de Eventos
```typescript
interface DomainEvent {
  id: string;
  type: string;
  aggregateId: string;
  aggregateType: string;
  data: any;
  metadata: {
    tenant_id: string;
    user_id: string;
    timestamp: string;
    correlation_id: string;
  };
}

class UserCreatedEvent implements DomainEvent {
  constructor(
    public readonly id: string,
    public readonly aggregateId: string,
    public readonly data: UserCreatedData,
    public readonly metadata: EventMetadata
  ) {}

  readonly type = 'USER_CREATED';
  readonly aggregateType = 'User';
}
```

### 7. Error Handling

#### Hierarquia de Erros
````mermaid
%%{init: {'theme':'forest'}}%%
graph TB
    A[ServiceError] --> B[ValidationError]
    A --> C[BusinessError]
    A --> D[DataError]
    A --> E[IntegrationError]
    A --> F[SystemError]

    B --> B1[SchemaValidationError]
    B --> B2[BusinessRuleError]
    
    C --> C1[DomainError]
    C --> C2[AuthorizationError]
    
    D --> D1[DatabaseError]
    D --> D2[CacheError]
    
    E --> E1[ThirdPartyError]
    E --> E2[MessageQueueError]
    
    F --> F1[ConfigurationError]
    F --> F2[InternalError]
````

#### Implementação de Erros
```typescript
abstract class ServiceError extends Error {
  abstract readonly code: string;
  abstract readonly statusCode: number;
  
  constructor(message: string, public readonly details?: any) {
    super(message);
    this.name = this.constructor.name;
  }
}

class BusinessError extends ServiceError {
  readonly statusCode = 400;
  
  constructor(
    public readonly code: string,
    message: string,
    details?: any
  ) {
    super(message, details);
  }
}

class DataError extends ServiceError {
  readonly statusCode = 500;
  readonly code = 'DATA_ERROR';
}
```

## Integração com Third-Party

### Gateway para Serviços Externos
````mermaid
%%{init: {'theme':'forest'}}%%
sequenceDiagram
    participant SL as Service Layer
    participant TG as Third Party Gateway
    participant CB as Circuit Breaker
    participant EX as External Service

    SL->>TG: Request External Service
    TG->>CB: Check Circuit State
    
    alt Circuit Closed
        CB->>EX: Forward Request
        EX-->>CB: Response
        CB-->>TG: Success Response
        TG-->>SL: Return Data
    else Circuit Open
        CB-->>TG: Circuit Open Error
        TG-->>SL: Fallback Response
    else Circuit Half-Open
        CB->>EX: Test Request
        alt Success
            EX-->>CB: Success Response
            CB->>CB: Close Circuit
            CB-->>TG: Success Response
        else Failure
            EX-->>CB: Error Response
            CB->>CB: Keep Circuit Open
            CB-->>TG: Fallback Response
        end
    end
````

### Circuit Breaker Implementation
```typescript
class CircuitBreaker {
  private state: 'CLOSED' | 'OPEN' | 'HALF_OPEN' = 'CLOSED';
  private failureCount = 0;
  private lastFailureTime?: Date;

  constructor(
    private readonly failureThreshold = 5,
    private readonly recoveryTimeout = 60000,
    private readonly monitoringPeriod = 10000
  ) {}

  async execute<T>(operation: () => Promise<T>): Promise<T> {
    if (this.state === 'OPEN') {
      if (this.shouldAttemptReset()) {
        this.state = 'HALF_OPEN';
      } else {
        throw new Error('Circuit breaker is OPEN');
      }
    }

    try {
      const result = await operation();
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }
}
```

## Logging e Monitoramento

### Estrutura de Logs
````mermaid
%%{init: {'theme':'forest'}}%%
graph TB
    subgraph "Log Types"
        A[Application Logs]
        B[Audit Logs]
        C[Performance Logs]
        D[Error Logs]
        E[Business Logs]
    end

    subgraph "Log Processing"
        F[Log Aggregator]
        G[Log Parser]
        H[Log Storage]
        I[Log Analytics]
    end

    A --> F
    B --> F
    C --> F
    D --> F
    E --> F
    F --> G
    G --> H
    H --> I
````

### Formato de Log Padronizado
```json
{
  "timestamp": "2024-03-20T10:00:00Z",
  "level": "INFO",
  "service": "user-service",
  "version": "1.2.3",
  "environment": "production",
  "tenant_id": "tenant-123",
  "user_id": "user-456",
  "request_id": "req-789",
  "correlation_id": "corr-012",
  "action": "CREATE_USER",
  "message": "Usuário criado com sucesso",
  "duration_ms": 150,
  "metadata": {
    "entity_id": "user-789",
    "ip": "192.168.1.1",
    "user_agent": "ServiceClient/1.0"
  }
}
```

## Métricas e KPIs

### Métricas de Performance
- **Throughput**: Requisições por segundo
- **Latência**: Tempo médio de resposta (P50, P95, P99)
- **Taxa de Erro**: Porcentagem de requisições com erro
- **Disponibilidade**: Uptime do serviço

### Métricas de Negócio
- **Conversão**: Taxa de sucesso por operação
- **Volume**: Número de operações por período
- **Qualidade**: Dados corretos vs incorretos

### Dashboard de Monitoramento
````mermaid
%%{init: {'theme':'forest'}}%%
graph TB
    subgraph "Monitoring Dashboard"
        A[Health Status]
        B[Performance Metrics]
        C[Business Metrics]
        D[Error Tracking]
        
        subgraph "Alerts"
            E[High Error Rate]
            F[High Latency]
            G[Resource Usage]
            H[Business Rules Violation]
        end
    end
````

## Testes

### Estratégia de Testes
````mermaid
%%{init: {'theme':'forest'}}%%
pyramid TB
    subgraph "Test Pyramid"
        A[Unit Tests] --> B[Integration Tests]
        B --> C[Contract Tests]
        C --> D[End-to-End Tests]
    end
````

### Tipos de Teste
- **Unit Tests**: Teste de componentes isolados
- **Integration Tests**: Teste de integração entre componentes
- **Contract Tests**: Teste de contratos de API
- **Performance Tests**: Teste de carga e stress
- **Security Tests**: Teste de vulnerabilidades

## Configuração e Deploy

### Configuração por Ambiente
```yaml
service:
  name: "user-service"
  version: "1.2.3"
  port: 3001
  
database:
  host: "${DB_HOST}"
  port: 5432
  name: "${DB_NAME}"
  user: "${DB_USER}"
  password: "${DB_PASSWORD}"
  pool_size: 20
  
cache:
  redis:
    host: "${REDIS_HOST}"
    port: 6379
    password: "${REDIS_PASSWORD}"
    ttl_default: 3600
    
messaging:
  kafka:
    brokers: ["${KAFKA_BROKER_1}", "${KAFKA_BROKER_2}"]
    client_id: "user-service"
    group_id: "user-service-group"
    
monitoring:
  metrics_enabled: true
  tracing_enabled: true
  log_level: "info"
```

### Estratégia de Deploy
````mermaid
%%{init: {'theme':'forest'}}%%
graph LR
    A[Code Push] --> B[Build]
    B --> C[Test]
    C --> D[Package]
    D --> E[Deploy Dev]
    E --> F[Deploy Staging]
    F --> G[Deploy Production]

    subgraph "Deploy Strategies"
        H[Blue/Green]
        I[Rolling Update]
        J[Canary]
    end
````

## Considerações de Segurança

### Proteções Implementadas
1. **Validação de Entrada**
   - Sanitização de dados
   - Validação de schema
   - Prevenção de injection

2. **Controle de Acesso**
   - Verificação de permissões
   - Isolamento de tenant
   - Auditoria de ações

3. **Proteção de Dados**
   - Criptografia de dados sensíveis
   - Mascaramento de logs
   - Backup seguro

### Políticas de Segurança
````mermaid
%%{init: {'theme':'forest'}}%%
graph TB
    subgraph "Security Layers"
        A[Input Validation]
        B[Authorization]
        C[Data Protection]
        D[Audit Logging]
        
        A --> B
        B --> C
        C --> D
    end
````

## Escalabilidade

### Estratégias de Escalabilidade
- **Horizontal Scaling**: Múltiplas instâncias do serviço
- **Vertical Scaling**: Aumento de recursos por instância
- **Database Sharding**: Distribuição de dados
- **Cache Distribution**: Cache distribuído

### Auto-scaling
````mermaid
%%{init: {'theme':'forest'}}%%
graph LR
    A[Metrics Collection] --> B[Scaling Decision]
    B --> C[Scale Up/Down]
    C --> D[Load Balancer Update]
    D --> E[Health Check]
    E --> A
````

## Conclusão

A Service Layer é o núcleo da lógica de negócio, projetada para ser:
- **Resiliente**: Tolerante a falhas e auto-recuperável
- **Escalável**: Capaz de crescer conforme a demanda
- **Maintível**: Fácil de manter e evoluir
- **Testável**: Cobertura completa de testes
- **Observável**: Monitoramento e logging detalhados
- **Segura**: Proteção em múltiplas camadas

Esta implementação garante que cada serviço seja independente, focado em seu domínio específico e capaz de evoluir sem impactar outros componentes do sistema.
