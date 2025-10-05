# Architecture Decision Records (ADRs)

> **Registro de decisões arquiteturais importantes do RPG System**

## 📋 O que são ADRs?

Architecture Decision Records (ADRs) são documentos que capturam decisões arquiteturais importantes, incluindo:
- **Contexto:** Por que a decisão foi necessária
- **Decisão:** O que foi decidido
- **Alternativas:** O que foi considerado e rejeitado
- **Consequências:** Impactos positivos e negativos

## 🎯 ADRs do RPG System

### [ADR 001: Arquitetura de Microserviços](./001-arquitetura-microservicos.md)
**Status:** ✅ Aceito | **Data:** 2025-10-05

**Resumo:** Decisão de adotar arquitetura de microserviços com 8 serviços independentes.

**Por quê?**
- Tecnologias heterogêneas (Node.js, Rust, Python)
- Escalabilidade independente
- Controle de custos por tier
- Fail gracefully

**Principais Serviços:**
- API Gateway (NestJS)
- Character Generator (NestJS + OpenAI)
- Battle Simulator (Go) ⚡ Mudado de Rust para Go
- Story Generator (Python + OpenAI)
- Dice Roller (Node.js + Redis)
- Notification Service (Node.js)
- Analytics Service (Python + TimescaleDB)
- Campaign Manager (NestJS)

**Decisão Go vs Rust:**
- Go oferece 80% da performance com 20% da complexidade
- Goroutines perfeitas para concorrência
- Mais fácil contratar e treinar desenvolvedores
- Deployment mais simples (binário único)

---

### [ADR 002: Estratégia de Dados](./002-estrategia-dados.md)
**Status:** ✅ Aceito | **Data:** 2025-10-05

**Resumo:** Database per Service com bancos especializados para cada caso de uso.

**Por quê?**
- Desacoplamento total entre serviços
- Bancos otimizados (PostgreSQL, MongoDB, Redis, TimescaleDB)
- Escalabilidade independente
- Migrations isoladas

**Mapeamento:**
- Character Generator → PostgreSQL (relacional, ACID)
- Battle Simulator → MongoDB (documentos, schema flexível)
- Story Generator → PostgreSQL (full-text search)
- Dice Roller → Redis (in-memory, baixa latência)
- Analytics → TimescaleDB (séries temporais)
- Campaign Manager → PostgreSQL (relacionamentos)

**Trade-offs:**
- ✅ Performance otimizada
- ✅ Desacoplamento
- ⚠️ Consistência eventual
- ⚠️ Queries cross-service mais complexas

---

### [ADR 003: Integração com IA](./003-integracao-ia.md)
**Status:** ✅ Aceito | **Data:** 2025-10-05

**Resumo:** Integração com OpenAI (GPT-4 e DALL-E) como diferencial competitivo.

**Por quê?**
- Diferenciação no mercado
- Valor percebido alto
- Justifica monetização
- Experiência do usuário superior

**Onde usar IA:**
1. **Character Generator:**
   - Interpreta linguagem natural
   - Gera stats otimizados
   - Cria backstory completa
   - Gera avatar (DALL-E)
   - Custo: ~$0.10/personagem

2. **Battle Simulator:**
   - Narra combates dinamicamente
   - Custo: ~$0.05/combate

3. **Story Generator:**
   - Gera missões, NPCs, histórias
   - Custo: ~$0.20/missão

4. **Dice Roller:**
   - Detecção de trapaça (ML próprio)
   - Custo: $0

**Controle de Custos:**
- Rate limiting por tier
- Cache de respostas
- Fallback para geração simples
- Monitoramento constante

**Custos por Tier:**
- Free: $0 (sem IA)
- Pro: ~$3/mês (100 gerações)
- Master: ~$10/mês (1000 gerações)

---

### [ADR 004: Event-Driven Architecture](./004-event-driven.md)
**Status:** 🚧 Planejado | **Data:** TBD

**Resumo:** Comunicação assíncrona via Kafka para desacoplamento temporal.

**Tópicos:**
- Padrões de eventos
- Saga pattern para transações distribuídas
- Event sourcing (futuro)
- CQRS (futuro)

---

## 📊 Visão Geral das Decisões

### Princípios Arquiteturais

1. **Single Responsibility**
   - Cada microserviço tem uma responsabilidade única

2. **Database per Service**
   - Cada serviço possui seu próprio banco de dados

3. **Event-Driven Communication**
   - Comunicação assíncrona via Kafka

4. **API Gateway Pattern**
   - Ponto único de entrada para clientes

5. **Fail Fast & Graceful Degradation**
   - Serviços falham rapidamente, sistema continua funcionando

6. **Observability First**
   - Logs, métricas e tracing desde o início

### Stack Tecnológico

| Camada | Tecnologias |
|--------|-------------|
| **Backend** | Node.js (NestJS), Go, Python |
| **IA** | OpenAI GPT-4, DALL-E 3 |
| **Databases** | PostgreSQL, MongoDB, Redis, TimescaleDB |
| **Messaging** | Apache Kafka |
| **Infra** | Docker, Kubernetes |
| **Observability** | Prometheus, Grafana, Jaeger |

### Justificativa do Modelo

```
┌─────────────────────────────────────────────────────────┐
│                  Por que Microserviços?                  │
├─────────────────────────────────────────────────────────┤
│ 1. Tecnologias diferentes (Node, Rust, Python)         │
│ 2. Escalabilidade independente                         │
│ 3. Controle de custos por tier                         │
│ 4. Fail gracefully                                     │
│ 5. Deploy independente                                 │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│                Por que Database per Service?             │
├─────────────────────────────────────────────────────────┤
│ 1. Bancos especializados (MongoDB, Redis, etc)         │
│ 2. Desacoplamento total                                │
│ 3. Escalabilidade independente                         │
│ 4. Migrations isoladas                                 │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│                    Por que IA?                           │
├─────────────────────────────────────────────────────────┤
│ 1. Diferenciação competitiva                           │
│ 2. Valor percebido alto (10-20x ROI)                  │
│ 3. Justifica monetização                               │
│ 4. Experiência superior                                │
└─────────────────────────────────────────────────────────┘
```

## 🎯 Roadmap de Implementação

### Fase 1: MVP (3 meses)
- ✅ ADR 001: Arquitetura de Microserviços
- ✅ ADR 002: Estratégia de Dados (PostgreSQL único)
- 🚧 API Gateway básico
- 🚧 Character Generator (sem IA)
- 🚧 Dice Roller básico

### Fase 2: Microserviços (3 meses)
- 🚧 Separar databases
- 🚧 Battle Simulator (Rust)
- 🚧 Story Generator (Python)
- 🚧 Kafka completo
- 🚧 ADR 004: Event-Driven

### Fase 3: IA (2 meses)
- ✅ ADR 003: Integração com IA
- 🚧 OpenAI no Character Generator
- 🚧 DALL-E para avatares
- 🚧 Narrativa de combates

### Fase 4: Escalabilidade (2 meses)
- 🚧 Kubernetes
- 🚧 Auto-scaling
- 🚧 Redis cache
- 🚧 Analytics Service

## 📚 Documentos Relacionados

- [📖 Arquitetura Completa](../ARCHITECTURE.md)
- [💰 Modelo de Negócio](../BUSINESS_MODEL.md)
- [🏠 README Principal](../../README.md)

## 🤝 Como Contribuir com ADRs

### Quando Criar uma ADR?

Crie uma ADR quando:
- ✅ Decisão afeta múltiplos serviços
- ✅ Decisão tem impacto significativo
- ✅ Decisão é difícil de reverter
- ✅ Decisão precisa ser justificada

**Não** crie ADR para:
- ❌ Decisões triviais
- ❌ Decisões locais (um serviço)
- ❌ Decisões facilmente reversíveis

### Template

Use o [TEMPLATE.md](./TEMPLATE.md) para criar novas ADRs.

### Processo

1. Crie branch: `adr/XXX-titulo-da-decisao`
2. Copie TEMPLATE.md para `XXX-titulo.md`
3. Preencha todas as seções
4. Discuta com o time
5. Abra PR
6. Após aprovação, merge e atualize este README

## 📊 Status das ADRs

| ADR | Título | Status | Data |
|-----|--------|--------|------|
| 001 | Arquitetura de Microserviços | ✅ Aceito | 2025-10-05 |
| 002 | Estratégia de Dados | ✅ Aceito | 2025-10-05 |
| 003 | Integração com IA | ✅ Aceito | 2025-10-05 |
| 004 | Event-Driven Architecture | 🚧 Planejado | TBD |
| 005 | Segurança e Autenticação | 📋 Proposto | TBD |
| 006 | Observabilidade | 📋 Proposto | TBD |

**Legenda:**
- ✅ Aceito: Decisão aprovada e implementada/em implementação
- 🚧 Planejado: Decisão planejada, aguardando escrita
- 📋 Proposto: Ideia de ADR, precisa discussão
- ❌ Rejeitado: Decisão considerada e rejeitada
- 🔄 Superseded: Substituída por outra ADR

---

<p align="center">
  <strong>ADRs são documentos vivos.</strong><br>
  Atualize conforme o sistema evolui.
</p>