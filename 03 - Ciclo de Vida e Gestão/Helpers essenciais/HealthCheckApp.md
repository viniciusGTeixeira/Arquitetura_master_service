# HealthCheckApp – Health Check Customizado no Laravel utilizando a lib spatie/laravel-health

## O que é?
O `HealthCheckApp` é um helper customizado para realizar verificações de saúde (health check) essenciais da aplicação Laravel, como banco de dados, cache e fila. Ele centraliza a lógica de checagem, permitindo fácil reutilização e manutenção.

## Como funciona?
- O método estático `HealthCheckApp::check()` executa testes de conexão com:
  - **Banco de dados**: Testa se a conexão está ativa.
  - **Cache**: Testa se é possível gravar, ler e remover um valor temporário.
  - **Fila**: Verifica se o driver padrão de fila está configurado.
- O resultado é um array indicando o status de cada serviço.
- O endpoint `/health` utiliza esse helper para retornar um JSON com o status geral e detalhado.

## Exemplo de resposta do endpoint
```json
{
  "status": "ok",
  "checks": {
    "database": "ok",
    "cache": "ok",
    "queue": "ok"
  },
  "timestamp": "2024-07-18T20:00:00-03:00"
}
```

## Por que usar um health check customizado?
- **Simplicidade**: Fácil de entender, modificar e expandir.
- **Controle total**: Você define exatamente o que será checado e como.
- **Sem dependências externas**: Não adiciona pacotes ao projeto.
- **Ideal para APIs, automações e monitoramento básico**.

## Alternativa recomendada: spatie/laravel-health
Se você deseja um painel visual, checks prontos para diversos serviços (Horizon, Redis, disco, jobs, ambiente, debug, etc) e integração fácil com ferramentas de monitoramento, a biblioteca [spatie/laravel-health](https://spatie.be/docs/laravel-health) é a melhor escolha.

### Vantagens da spatie/laravel-health
- Painel web moderno e responsivo (igual ao da imagem de exemplo)
- Checks prontos para banco, cache, filas, disco, jobs, ambiente, debug, Redis, Horizon, Schedule, etc
- Extensível: crie checks customizados facilmente
- Integração com notificações, logs e alertas
- Proteção de rota por autenticação ou IP

### Quando usar cada abordagem?
- **Customizado (HealthCheckApp)**: Quando precisa de algo simples, leve, sem dependências e fácil de adaptar ao seu contexto.
- **spatie/laravel-health**: Quando precisa de um painel visual, monitoramento avançado, integração com times de DevOps/SRE, ou quer evitar reinventar a roda.

## Como migrar para o pacote spatie/laravel-health
1. Instale: `composer require spatie/laravel-health`
2. Publique as configs/views: `php artisan vendor:publish --tag=health-config --tag=health-views`
3. Configure os checks em `config/health.php`
4. Acesse `/health` para ver o painel visual

---

**Resumo:**
- Use o helper customizado para soluções simples e sob medida.
- Use o pacote Spatie para monitoramento profissional, visual e escalável. 