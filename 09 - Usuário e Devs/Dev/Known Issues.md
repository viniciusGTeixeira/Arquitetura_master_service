# Known Issues
## Problemas Conhecidos da Estrutura

---

## 1. Lista de Problemas
| Problema                        | Causa Possível                | Impacto                | Possível Solução           |
|---------------------------------|-------------------------------|------------------------|----------------------------|
| Latência alta em horários de pico| Sobrecarga no banco/Kafka     | Lentidão para usuários | Otimizar queries, escalar  |
| Falha de integração com terceiros| Timeout ou API externa instável| Erro em funcionalidades| Implementar retry/fallback |
| Erros intermitentes de autenticação| Sincronismo de clock, Keycloak| Login falha ocasional | Sincronizar clocks, monitorar|
| Consumo elevado de memória      | Vazamento de memória em serviço| Risco de crash         | Revisar código, profiling  |
| Logs excessivos                 | Falta de rotação/configuração | Disco cheio            | Configurar rotação/logrotate|

---

## 2. Observações
- A lista deve ser revisada e atualizada a cada release
- Problemas críticos devem ser priorizados no backlog