# 🐳 Docker Swarm - Orquestração de Containers

![Docker](https://img.shields.io/badge/Docker-20.10+-2496ED?style=flat-square&logo=docker&logoColor=white)
![Docker Swarm](https://img.shields.io/badge/Docker%20Swarm-Orchestrator-2496ED?style=flat-square&logo=docker)
![License](https://img.shields.io/badge/License-Educational-yellow?style=flat-square)

## 📋 Visão Geral

Projeto educacional que demonstra a implementação de **orquestração de containers** utilizando **Docker Swarm**. Implementa uma arquitetura completa de cluster com múltiplos serviços, balanceamento de carga, replicação e alta disponibilidade através de uma aplicação de votação distribuída.

> **🎓 Projeto Educacional**: Desenvolvido com base no curso **"Docker Swarm: Orquestrador de containers"** da **Alura**.

---

## 🎯 Objetivo do Projeto

Este projeto tem como objetivos:
- ✅ **Demonstrar orquestração** de containers com Docker Swarm
- ✅ **Implementar cluster distribuído** com managers e workers
- ✅ **Aplicar conceitos** de alta disponibilidade e escalabilidade
- ✅ **Gerenciar serviços** replicados e globais
- ✅ **Configurar redes overlay** para comunicação entre containers
- ✅ **Deploy de stack completa** com Docker Compose

---

## 🏗️ Arquitetura da Aplicação

### 📊 Aplicação de Votação Distribuída

A aplicação implementa um sistema de votação completo com as seguintes camadas:

```
┌─────────────────────────────────────────────────────────────┐
│                    DOCKER SWARM CLUSTER                     │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────┐                            ┌─────────────┐ │
│  │   Manager   │◄──────────────────────────►│   Worker    │ │
│  │    Node     │         Overlay Network    │    Node     │ │
│  └─────────────┘                            └─────────────┘ │
│         │                                           │       │
│         ▼                                           ▼       │
│  ┌──────────────────────────────────────────────────────┐   │
│  │              FRONTEND NETWORK                        │   │
│  ├──────────────────────────────────────────────────────┤   │
│  │  ┌──────────┐      ┌──────────┐                      │   │
│  │  │   Vote   │──────│  Redis   │                      │   │
│  │  │ Service  │      │  Cache   │                      │   │
│  │  │  :5000   │      │          │                      │   │
│  │  └──────────┘      └──────────┘                      │   │
│  │  (2 replicas)      (1 replica)                       │   │
│  └──────────────────────────────────────────────────────┘   │
│                            │                                │
│                            ▼                                │
│                    ┌──────────────┐                         │
│                    │    Worker    │                         │
│                    │   Service    │                         │
│                    │ (Processing) │                         │
│                    └──────────────┘                         │
│                            │                                │
│                            ▼                                │
│  ┌──────────────────────────────────────────────────────┐   │
│  │              BACKEND NETWORK                         │   │
│  ├──────────────────────────────────────────────────────┤   │
│  │  ┌──────────┐      ┌──────────┐                      │   │ 
│  │  │  Result  │──────│PostgreSQL│                      │   │
│  │  │ Service  │      │    DB    │                      │   │
│  │  │  :5001   │      │          │                      │   │
│  │  └──────────┘      └──────────┘                      │   │
│  │  (1 replica)       (1 replica)                       │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐   │
│  │              VISUALIZER                              │   │
│  │        (Monitoring Dashboard :8080)                  │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

### 🔧 Componentes da Stack

#### **Frontend Layer**
- **Vote Service** (Python/Flask)
  - Interface de votação para usuários
  - 2 réplicas para alta disponibilidade
  - Porta: 5000
  - Conectado ao Redis

- **Redis Cache**
  - Armazena votos temporariamente
  - 1 réplica
  - Banco de dados em memória

#### **Processing Layer**
- **Worker Service** (.NET)
  - Processa votos do Redis
  - Persiste no PostgreSQL
  - 1 réplica
  - Executado apenas em nós workers

#### **Backend Layer**
- **Result Service** (Node.js)
  - Exibe resultados da votação
  - 1 réplica
  - Porta: 5001
  - Conectado ao PostgreSQL

- **PostgreSQL Database**
  - Armazena resultados finais
  - 1 réplica
  - Volume persistente
  - Executado apenas em nós managers

#### **Monitoring Layer**
- **Visualizer**
  - Dashboard visual do cluster
  - Porta: 8080
  - Executado apenas em nós managers

---

## 🛠️ Tecnologias e Recursos

### 🐳 Container Orchestration

| Componente | Tecnologia | Versão | Propósito |
|-----------|------------|--------|-----------|
| **Orchestrator** | Docker Swarm | 20.10+ | Orquestração de containers |
| **Container Runtime** | Docker Engine | 20.10+ | Execução de containers |
| **Compose** | Docker Compose | v3 | Definição de serviços |
| **Machine** | Docker Machine | 0.16+ | Criação de hosts remotos |

### 📦 Serviços da Stack

| Serviço | Imagem | Linguagem | Réplicas | Portas |
|---------|--------|-----------|----------|--------|
| **vote** | dockersamples/examplevotingapp_vote:before | Python/Flask | 2 | 5000:80 |
| **result** | dockersamples/examplevotingapp_result:before | Node.js | 1 | 5001:80 |
| **worker** | dockersamples/examplevotingapp_worker | .NET | 1 | - |
| **redis** | redis:alpine | - | 1 | 6379 |
| **db** | postgres:9.4 | - | 1 | 5432 |
| **visualizer** | dockersamples/visualizer:stable | - | 1 | 8080:8080 |

### 🌐 Redes Overlay

```yaml
networks:
  frontend:    # Comunicação Vote ↔ Redis
  backend:     # Comunicação Result ↔ PostgreSQL
```

**Driver**: Overlay (comunicação multi-host)

### 💾 Volumes Persistentes

```yaml
volumes:
  db-data:     # Dados do PostgreSQL
```

---

## 📁 Estrutura do Projeto

```
docker-swarm-config-main/
│
├── 📄 docker-compose.yml       # Definição completa da stack
├── 📄 README.md               # Documentação original
└── 📄 README_TECNICO.md       # Este arquivo (documentação técnica)
```

### 📊 Métricas do Projeto

- **Total de serviços**: 6
- **Total de réplicas**: 8 (distribuídas)
- **Redes overlay**: 2
- **Volumes persistentes**: 1
- **Linhas de código**: ~84 (docker-compose.yml)
- **Tamanho do projeto**: ~1.3 KB

---

## 🚀 Configuração e Deploy

### 📋 Pré-requisitos

#### Obrigatórios
- ✅ **Docker Engine 20.10+**
  ```bash
  docker --version
  ```
- ✅ **Docker Compose v3+**
  ```bash
  docker-compose --version
  ```

#### Opcionais (para cluster multi-host)
- 📦 **Docker Machine 0.16+**
  ```bash
  docker-machine --version
  ```
- 🖥️ **VirtualBox** ou outro driver de virtualização
- ☁️ **Provedor cloud** (AWS, Azure, DigitalOcean)

#### Requisitos de Sistema
- **RAM**: Mínimo 4GB por nó
- **CPU**: 2 cores por nó (recomendado)
- **Disco**: 10GB disponível
- **Portas necessárias**: 2377, 7946, 4789, 5000, 5001, 8080

---

## 🔧 Instalação Passo a Passo

### 🎯 Cenário 1: Cluster Local (Single Node)

#### 1. Inicializar o Swarm
```bash
# Iniciar modo Swarm
docker swarm init

# Verificar status
docker node ls
```

**Saída esperada**:
```
ID                            HOSTNAME   STATUS    AVAILABILITY   MANAGER STATUS
abc123def456ghi789jkl012 *   manager1   Ready     Active         Leader
```

#### 2. Deploy da Stack
```bash
# Deploy da aplicação completa
docker stack deploy -c docker-compose.yml votingapp

# Verificar serviços
docker stack services votingapp
```

**Saída esperada**:
```
ID            NAME                MODE        REPLICAS   IMAGE
xyz1          votingapp_vote      replicated  2/2        dockersamples/examplevotingapp_vote:before
xyz2          votingapp_result    replicated  1/1        dockersamples/examplevotingapp_result:before
xyz3          votingapp_worker    replicated  1/1        dockersamples/examplevotingapp_worker
xyz4          votingapp_redis     replicated  1/1        redis:alpine
xyz5          votingapp_db        replicated  1/1        postgres:9.4
xyz6          votingapp_visualizer replicated 1/1        dockersamples/visualizer:stable
```

#### 3. Acessar as Aplicações
- **Vote** (Interface de Votação): http://localhost:5000
- **Result** (Resultados): http://localhost:5001
- **Visualizer** (Dashboard do Cluster): http://localhost:8080

---

### 🌐 Cenário 2: Cluster Multi-Node (Produção)

#### 1. Criar Nós com Docker Machine

##### Manager Node
```bash
# Criar nó manager
docker-machine create \
  --driver virtualbox \
  --virtualbox-memory 2048 \
  --virtualbox-cpu-count 2 \
  manager1

# Obter IP do manager
docker-machine ip manager1
# Exemplo: 192.168.99.100
```

##### Worker Nodes
```bash
# Criar worker 1
docker-machine create \
  --driver virtualbox \
  --virtualbox-memory 2048 \
  worker1

# Criar worker 2
docker-machine create \
  --driver virtualbox \
  --virtualbox-memory 2048 \
  worker2

# Listar máquinas criadas
docker-machine ls
```

#### 2. Inicializar Swarm no Manager

```bash
# Conectar ao manager
eval $(docker-machine env manager1)

# Inicializar Swarm
docker swarm init --advertise-addr $(docker-machine ip manager1)
```

**Saída** (guarde o comando de join):
```
Swarm initialized: current node (abc123) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-xyz... 192.168.99.100:2377
```

#### 3. Adicionar Workers ao Cluster

```bash
# Conectar ao worker1
eval $(docker-machine env worker1)

# Join no cluster
docker swarm join --token SWMTKN-1-xyz... 192.168.99.100:2377

# Repetir para worker2
eval $(docker-machine env worker2)
docker swarm join --token SWMTKN-1-xyz... 192.168.99.100:2377
```

#### 4. Verificar Cluster

```bash
# Voltar ao manager
eval $(docker-machine env manager1)

# Listar nós
docker node ls
```

**Saída esperada**:
```
ID                            HOSTNAME   STATUS    AVAILABILITY   MANAGER STATUS
abc123def456ghi789jkl012 *   manager1   Ready     Active         Leader
def456ghi789jkl012mno345     worker1    Ready     Active         
ghi789jkl012mno345pqr678     worker2    Ready     Active         
```

#### 5. Deploy da Stack

```bash
# Deploy (no manager)
docker stack deploy -c docker-compose.yml votingapp

# Monitorar deploy
watch docker stack ps votingapp
```

#### 6. Acessar Aplicações

```bash
# Obter IP do manager
MANAGER_IP=$(docker-machine ip manager1)

# URLs
echo "Vote:       http://$MANAGER_IP:5000"
echo "Result:     http://$MANAGER_IP:5001"
echo "Visualizer: http://$MANAGER_IP:8080"
```

---

## 🔍 Análise Detalhada dos Serviços

### 🗳️ Vote Service

```yaml
vote:
  image: dockersamples/examplevotingapp_vote:before
  ports:
    - 5000:80
  networks:
    - frontend
  depends_on:
    - redis
  deploy:
    replicas: 2
    restart_policy:
      condition: on-failure
```

**Características**:
- ✅ **2 réplicas** para alta disponibilidade
- ✅ **Load balancing automático** entre réplicas
- ✅ **Restart automático** em caso de falha
- ✅ **Dependência**: Redis deve estar disponível

**Teste**:
```bash
# Verificar réplicas
docker service ps votingapp_vote

# Escalar manualmente
docker service scale votingapp_vote=5
```

---

### 📊 Result Service

```yaml
result:
  image: dockersamples/examplevotingapp_result:before
  ports:
    - 5001:80
  networks:
    - backend
  depends_on:
    - db
  deploy:
    replicas: 1
    restart_policy:
      condition: on-failure
```

**Características**:
- ✅ **1 réplica** (single point)
- ✅ **Conectado ao PostgreSQL** via backend network
- ✅ **Restart automático** em caso de falha

---

### ⚙️ Worker Service

```yaml
worker:
  image: dockersamples/examplevotingapp_worker
  networks:
    - frontend
    - backend
  depends_on:
    - db
    - redis
  deploy:
    mode: replicated
    replicas: 1
    labels: [APP=VOTING]
    restart_policy:
      condition: on-failure
    placement:
      constraints: [node.role == worker]
```

**Características**:
- ✅ **Bridge entre frontend e backend**
- ✅ **Placement constraint**: executa apenas em workers
- ✅ **Label customizada**: APP=VOTING
- ✅ **Processa votos**: Redis → PostgreSQL

---

### 🗄️ Database Service (PostgreSQL)

```yaml
db:
  image: postgres:9.4
  volumes:
    - db-data:/var/lib/postgresql/data
  networks:
    - backend
  deploy:
    placement:
      constraints: [node.role == manager]
  environment:
    POSTGRES_HOST_AUTH_METHOD: trust
```

**Características**:
- ✅ **Placement constraint**: executa apenas em managers
- ✅ **Volume persistente**: dados sobrevivem a reinicializações
- ✅ **Autenticação simplificada**: modo trust (não recomendado para produção)

**⚠️ Nota de Segurança**:
Para produção, configure senha:
```yaml
environment:
  POSTGRES_PASSWORD: senhaSegura123
  POSTGRES_USER: votingapp
  POSTGRES_DB: votes
```

---

### 📦 Redis Cache

```yaml
redis:
  image: redis:alpine
  networks:
    - frontend
  deploy:
    replicas: 1
    restart_policy:
      condition: on-failure
```

**Características**:
- ✅ **Imagem Alpine**: menor footprint (5MB)
- ✅ **Armazenamento em memória**: alta performance
- ✅ **Restart automático** em caso de falha

---

### 👁️ Visualizer

```yaml
visualizer:
  image: dockersamples/visualizer:stable
  ports:
    - 8080:8080
  stop_grace_period: 1m30s
  volumes:
    - "/var/run/docker.sock:/var/run/docker.sock"
  deploy:
    placement:
      constraints: [node.role == manager]
```

**Características**:
- ✅ **Dashboard visual** do cluster
- ✅ **Acesso ao Docker socket**: comunicação com daemon
- ✅ **Graceful shutdown**: 90 segundos para finalizar
- ✅ **Placement constraint**: apenas em managers

**⚠️ Nota de Segurança**:
O volume do socket Docker concede acesso privilegiado. Não expor em produção.

---

## 📊 Gerenciamento do Cluster

### 🔍 Comandos de Monitoramento

#### Verificar Status Geral
```bash
# Status dos nós
docker node ls

# Inspecionar nó específico
docker node inspect manager1 --pretty

# Serviços da stack
docker stack services votingapp

# Tasks (containers) em execução
docker stack ps votingapp
```

#### Logs e Debugging
```bash
# Logs de um serviço
docker service logs votingapp_vote

# Logs em tempo real
docker service logs -f votingapp_vote

# Logs de um container específico
docker logs <container_id>
```

#### Métricas de Performance
```bash
# Estatísticas de containers
docker stats

# Inspecionar serviço
docker service inspect votingapp_vote --pretty
```

---

### ⚖️ Escalabilidade

#### Escalar Serviços
```bash
# Aumentar réplicas do vote
docker service scale votingapp_vote=5

# Múltiplos serviços simultaneamente
docker service scale votingapp_vote=5 votingapp_result=3

# Verificar distribuição
docker service ps votingapp_vote
```

#### Modo Global vs Replicado
```yaml
# Modo Replicado (padrão)
deploy:
  mode: replicated
  replicas: 3

# Modo Global (1 réplica por nó)
deploy:
  mode: global
```

---

### 🔄 Atualizações Rolling

```bash
# Atualizar imagem do serviço
docker service update \
  --image dockersamples/examplevotingapp_vote:after \
  votingapp_vote

# Rollback em caso de problema
docker service rollback votingapp_vote

# Atualização com configuração customizada
docker service update \
  --update-parallelism 2 \
  --update-delay 10s \
  --update-failure-action rollback \
  --image nova-imagem:tag \
  votingapp_vote
```

---

## 🌐 Redes Overlay

### Conceito

As **redes overlay** permitem comunicação entre containers em diferentes hosts físicos de forma transparente.

### Redes Configuradas

#### Frontend Network
```yaml
networks:
  frontend:
```

**Conecta**:
- Vote Service ↔ Redis
- Worker ↔ Redis

**Propósito**: Isolamento da camada de apresentação

#### Backend Network
```yaml
networks:
  backend:
```

**Conecta**:
- Result Service ↔ PostgreSQL
- Worker ↔ PostgreSQL

**Propósito**: Isolamento da camada de dados

### Comandos de Rede

```bash
# Listar redes
docker network ls

# Inspecionar rede
docker network inspect votingapp_frontend

# Criar rede overlay customizada
docker network create \
  --driver overlay \
  --attachable \
  custom-network
```

---

## 🔐 Segurança e Boas Práticas

### ✅ Implementadas no Projeto

1. **Isolamento de Rede**
   - Frontend e Backend separados
   - Worker como único bridge entre camadas

2. **Placement Constraints**
   - Database apenas em managers (mais seguro)
   - Worker apenas em workers (isolamento)

3. **Restart Policies**
   - Todos os serviços com restart automático
   - Condition: on-failure

4. **Resource Limits** (pode adicionar)
   ```yaml
   deploy:
     resources:
       limits:
         cpus: '0.5'
         memory: 512M
       reservations:
         cpus: '0.25'
         memory: 256M
   ```

### ⚠️ Melhorias Recomendadas para Produção

1. **Secrets Management**
   ```bash
   # Criar secret
   echo "senhaSegura123" | docker secret create db_password -
   
   # Usar no serviço
   deploy:
     secrets:
       - db_password
   ```

2. **Health Checks**
   ```yaml
   healthcheck:
     test: ["CMD", "curl", "-f", "http://localhost/health"]
     interval: 30s
     timeout: 10s
     retries: 3
   ```

3. **Logging Centralizado**
   ```yaml
   logging:
     driver: "json-file"
     options:
       max-size: "10m"
       max-file: "3"
   ```

4. **Backup Automático**
   ```bash
   # Script de backup do PostgreSQL
   docker exec votingapp_db pg_dump -U postgres votes > backup.sql
   ```

---

## 🐛 Troubleshooting

### ❌ Problema: Serviço não inicia

**Sintomas**:
```bash
docker service ps votingapp_vote
# Status: Failed
```

**Diagnóstico**:
```bash
# Ver logs do serviço
docker service logs votingapp_vote

# Ver tasks com falha
docker service ps votingapp_vote --no-trunc
```

**Soluções Comuns**:
1. Imagem não encontrada → Verificar nome da imagem
2. Porta em uso → Mudar porta de binding
3. Dependência não disponível → Verificar ordem de deploy

---

### ❌ Problema: Nó não se conecta ao cluster

**Sintomas**:
```bash
docker swarm join --token ...
# Error: could not connect
```

**Diagnóstico**:
```bash
# Verificar conectividade
ping <manager-ip>

# Verificar portas abertas
telnet <manager-ip> 2377
```

**Soluções**:
1. Firewall bloqueando portas → Abrir 2377, 7946, 4789
2. IP incorreto → Usar `--advertise-addr` correto
3. Token expirado → Gerar novo token

---

### ❌ Problema: Volume não persiste dados

**Sintomas**:
Dados do PostgreSQL perdidos após restart

**Diagnóstico**:
```bash
# Verificar volume
docker volume ls
docker volume inspect votingapp_db-data
```

**Soluções**:
1. Volume não foi criado → Verificar definição no compose
2. Permissões incorretas → Ajustar permissões do host
3. Volume em nó diferente → Usar volume driver distribuído

---

## 🎓 Conceitos Abordados

### ✅ Docker Swarm Fundamentals

- **Cluster Management**: Gerenciar múltiplos hosts como um único sistema
- **Service Discovery**: Serviços encontram-se automaticamente
- **Load Balancing**: Distribuição automática de tráfego
- **Rolling Updates**: Atualizações sem downtime
- **Declarative Model**: Definir estado desejado, Swarm mantém

### ✅ Tipos de Nós

#### Manager Nodes
- **Responsabilidades**:
  - Gerenciar estado do cluster
  - Agendar serviços
  - Servir API do Swarm
- **Recomendações**:
  - Quantidade ímpar (3, 5, 7)
  - Máximo 7 managers
  - Mínimo 3 para produção

#### Worker Nodes
- **Responsabilidades**:
  - Executar containers
  - Reportar status ao manager
- **Recomendações**:
  - Sem limite de quantidade
  - Podem ser promovidos a managers

### ✅ Tipos de Serviços

#### Replicated (Replicado)
```yaml
deploy:
  mode: replicated
  replicas: 3
```
- **Uso**: Aplicações stateless
- **Exemplo**: Web servers, APIs

#### Global
```yaml
deploy:
  mode: global
```
- **Uso**: Daemons, agentes
- **Exemplo**: Monitoring, logging agents

---

## 📚 Recursos de Aprendizado

### 📖 Documentação Oficial
- [Docker Swarm Documentation](https://docs.docker.com/engine/swarm/)
- [Docker Compose v3 Reference](https://docs.docker.com/compose/compose-file/compose-file-v3/)
- [Docker Machine Documentation](https://docs.docker.com/machine/)
- [Docker Networking](https://docs.docker.com/network/)

### 🎓 Curso de Referência
- **[Docker Swarm: Orquestrador de containers](https://www.alura.com.br/)** - Alura
  - Gerenciamento de clusters
  - Docker Machine para criação de hosts
  - Managers, workers e serviços
  - Replicação e deploy de stacks
  - Redes overlay customizadas
  - Estratégias de deployment

### 📺 Recursos Adicionais
- [Play with Docker](https://labs.play-with-docker.com/) - Ambiente online gratuito
- [Docker Samples](https://github.com/dockersamples) - Exemplos oficiais
- [Docker Swarm Visualizer](https://github.com/dockersamples/docker-swarm-visualizer)

---

## 🔄 Comandos Essenciais

### Swarm Management
```bash
# Inicializar Swarm
docker swarm init

# Join como worker
docker swarm join --token SWMTKN-1-...

# Join como manager
docker swarm join-token manager

# Listar nós
docker node ls

# Promover worker a manager
docker node promote <node-id>

# Remover nó do cluster
docker node rm <node-id>

# Deixar o swarm (do nó)
docker swarm leave --force
```

### Stack Management
```bash
# Deploy stack
docker stack deploy -c docker-compose.yml votingapp

# Listar stacks
docker stack ls

# Serviços da stack
docker stack services votingapp

# Tasks da stack
docker stack ps votingapp

# Remover stack
docker stack rm votingapp
```

### Service Management
```bash
# Listar serviços
docker service ls

# Inspecionar serviço
docker service inspect votingapp_vote

# Logs do serviço
docker service logs votingapp_vote

# Escalar serviço
docker service scale votingapp_vote=5

# Atualizar serviço
docker service update --image nova-imagem votingapp_vote

# Remover serviço
docker service rm votingapp_vote
```

---

## 🗺️ Roadmap de Melhorias

### 🎯 Curto Prazo
- [x] Implementar stack básica com 6 serviços
- [x] Configurar redes overlay
- [x] Adicionar visualizador de cluster
- [ ] Implementar health checks
- [ ] Adicionar resource limits

### 🎯 Médio Prazo
- [ ] Integrar Docker Secrets
- [ ] Implementar CI/CD pipeline
- [ ] Adicionar monitoring com Prometheus
- [ ] Configurar logging centralizado (ELK)
- [ ] Implementar backup automatizado

### 🎯 Longo Prazo
- [ ] Migrar para Kubernetes (comparação)
- [ ] Implementar service mesh (Traefik)
- [ ] Adicionar auto-scaling
- [ ] Multi-cloud deployment
- [ ] Disaster recovery plan

---

## 📝 Notas Importantes

### ⚠️ Avisos
- 🔴 **Projeto Educacional**: Configurações simplificadas para aprendizado
- 🟡 **PostgreSQL Trust Mode**: Não usar em produção
- 🟠 **Docker Socket Exposto**: Risco de segurança no Visualizer

### ✅ Pontos Fortes
- ✔️ Arquitetura completa de referência
- ✔️ Demonstra conceitos fundamentais do Swarm
- ✔️ Fácil de replicar e experimentar
- ✔️ Visualização clara do cluster

### 📦 Informações Técnicas
- **Versão do Compose**: 3
- **Total de Serviços**: 6
- **Total de Réplicas**: 8
- **Redes**: 2 (frontend, backend)
- **Volumes**: 1 (db-data)

---

## 📄 Licença

Projeto educacional baseado em exemplos oficiais do Docker para fins de aprendizado.

---

## 🤝 Suporte e Contato

Para dúvidas sobre implementação:
- 📧 Documentação do Docker Swarm
- 💬 Docker Community Forums
- 📚 Alura - Plataforma de Cursos

---

**Desenvolvido com Docker Swarm** 🐳 | **Orquestração de Containers**  
**Baseado no curso**: [Docker Swarm: Orquestrador de containers - [Alura](https://www.alura.com.br/) 🎓  
*Última atualização: Abril 2024*
