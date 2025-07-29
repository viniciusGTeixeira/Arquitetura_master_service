# Deployment Diagram
## Diagrama de Deploy e Infraestrutura

---

## 1. Visão Geral
Este documento apresenta a arquitetura de infraestrutura do sistema, destacando componentes em nuvem, containers, redes e balanceadores.

---

## 2. Diagrama Mermaid
````mermaid
%%{init: {'theme':'forest'}}%%
graph TD
    subgraph Cloud [Cloud Provider]
        LB[Load Balancer]
        FW[Firewall]
        DNS[DNS]
        LB --> FW
        FW --> API["Master Layer / API Gateway / Podman || Docker"]
        API --> SVC["Service Layer / Podman or Docker"]
        API --> KC["Keycloak / Podman or Docker"]
        SVC --> DB[(PostgreSQL/MySQL)]
        SVC --> KAFKA[(Kafka)]
        API --> MON[(Prometheus/Grafana)]
    end
    DNS --> LB
    API --> EXT["Third-Party APIs"]

````

---

## 3. Componentes
- **Cloud Provider**: AWS
- **Load Balancer**: Distribui requisições entre instâncias
- **Firewall**: Protege a rede interna
- **Containers (Podman || Docker)**: Isolamento dos serviços (API, Service Layer, Keycloak)
- **Banco de Dados**: PostgreSQL/MySQL
- **Mensageria**: Kafka
- **Monitoramento**: Prometheus, Grafana
- **APIs Externas**: Integrações com terceiros

---

## 4. Observações
- Todos os serviços são orquestrados via Podman || Docker Compose/Kubernetes
- Rede interna isolada para comunicação entre containers
- Acesso externo apenas via Load Balancer e Firewall
- Backups automáticos e monitoramento contínuo