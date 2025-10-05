# ADR 001: Arquitetura de Microserviços

- **Status:** Aceito
- **Data:** 2025-10-05
- **Autor:** Laura Fumagalli
- **Decisores:** Equipe de Arquitetura

## Contexto

O RPG System precisa de uma arquitetura que suporte:

1. **Funcionalidades Heterogêneas:**
   - Geração de personagens com IA (processamento pesado, 10-30s)
   - Simulação de combates (CPU-intensive)
   - Rolagem de dados (tempo real, alta frequência)
   - Notificações (infraestrutura externa)
   - Analytics (big data, batch processing)

2. **Tecnologias Diferentes:**
   - Node.js para APIs e tempo real
   - Rust para performance crítica (combates)
   - Python para IA e análise de dados

3. **Escalabilidade Independente:**
   - Dice Roller: milhares de requests/segundo
   - Story Generator: dezenas de requests/hora
   - Custos diferentes (IA é cara)

4. **Modelo de Negócio:**
   - Free tier: sem IA, funcionalidades básicas
   - Pro tier: IA moderada
   - Master tier: IA intensiva
   - Precisa desligar serviços por tier

## Decisão

Adotar **arquitetura de microserviços** com os seguintes serviços:

### Microserviços Principais

1. **API Gateway** (NestJS)
   - Autenticação e autorização
   - Rate limiting por tier
   - Roteamento para microserviços

2. **Character Generator Service** (NestJS + OpenAI)
   - Gera personagens com IA
   - Integração GPT-4 e DALL-E
   - PostgreSQL próprio

3. **Battle Simulator Service** (Go)
   - Simula combates (performance e concorrência)
   - MongoDB próprio
   - Narrativa com IA

4. **Story Generator Service** (Python + OpenAI)
   - Gera histórias, NPCs, missões
   - PostgreSQL próprio
   - Rate limiting agressivo (IA cara)

5. **Dice Roller Service** (Node.js)
   - Rolagem tempo real (WebSocket)
   - Redis para cache
   - Detecção de trapaça

6. **Notification Service** (Node.js)
   - Email, push, WebSocket
   - Pode falhar isoladamente
   - Sem database próprio

7. **Analytics Service** (Python)
   - Métricas e insights
   - TimescaleDB
   - Processamento batch

8. **Campaign Manager Service** (NestJS)
   - Gerencia campanhas e sessões
   - PostgreSQL próprio
   - Domínio diferente

### Comunicação

- **Síncrona (REST):** Cliente → API Gateway → Microserviços
- **Assíncrona (Kafka):** Microserviços ↔ Microserviços

### Princípios

1. **Database per Service:** Cada serviço tem seu próprio banco
2. **Event-Driven:** Comunicação via Kafka para desacoplamento
3. **Stateless:** Todos os serviços são stateless
4. **API Gateway Pattern:** Ponto único de entrada
5. **Fail Fast:** Serviços falham rapidamente

## Alternativas Consideradas

### Alternativa 1: Monolito Modular

**Estrutura:**
```
rpg-system-api/
├── modules/
│   ├── characters/
│   ├── battles/
│   ├── stories/
│   └── dice/
└── shared/
```

**Vantagens:**
- ✅ Simples de desenvolver
- ✅ Fácil de debugar
- ✅ Sem overhead de rede

**Desvantagens:**
- ❌ Não pode usar tecnologias diferentes (Rust, Python)
- ❌ Escala tudo junto (desperdício)
- ❌ Não pode desligar funcionalidades por tier
- ❌ Deploy de tudo junto (risco)
- ❌ IA cara afeta todo sistema

**Por que rejeitamos:**
- Go para combates é 10x mais rápido que Node.js
- Python tem melhores libs de IA
- Não conseguimos controlar custos por tier
- Não podemos usar tecnologias especializadas

### Alternativa 2: Serverless (AWS Lambda)

**Estrutura:**
```
Lambda Functions:
├── character-generator
├── battle-simulator
├── story-generator
└── dice-roller
```

**Vantagens:**
- ✅ Escala automático
- ✅ Paga por uso
- ✅ Sem gerenciamento de infra

**Desvantagens:**
- ❌ Cold start (latência)
- ❌ Limite de tempo (15min)
- ❌ Difícil para WebSocket (dice roller)
- ❌ Vendor lock-in (AWS)
- ❌ Custo imprevisível

**Por que rejeitamos:**
- Dice Roller precisa WebSocket (tempo real)
- Battle Simulator pode demorar > 15min
- Queremos controle de custos

### Alternativa 3: Microserviços + Service Mesh

**Estrutura:**
```
Microserviços + Istio/Linkerd
```

**Vantagens:**
- ✅ Observabilidade avançada
- ✅ Traffic management
- ✅ Security (mTLS)

**Desvantagens:**
- ❌ Complexidade alta
- ❌ Overhead de aprendizado
- ❌ Overkill para MVP

**Por que rejeitamos:**
- Complexidade desnecessária para início
- Pode adicionar depois se necessário

---

## Decisão Técnica: Go vs Rust para Battle Simulator

### Por que Go (escolhido) ✅

**Performance:**
- 10x mais rápido que Node.js
- Suficiente para simular combates (< 100ms por round)
- GC moderno e eficiente

**Concorrência:**
- Goroutines são perfeitas para simular múltiplos combates
- Channels facilitam comunicação
- Scheduler eficiente

**Desenvolvimento:**
- Curva de aprendizado suave
- Código simples e legível
- Compile time rápido

**Operacional:**
- Binário único (fácil deploy)
- Cross-compilation simples
- Footprint pequeno

**Ecossistema:**
- gRPC nativo
- Kubernetes é escrito em Go
- Muitas libs de microserviços

**Time:**
- Mais fácil contratar devs Go
- Onboarding rápido (1-2 semanas)

### Por que NÃO Rust ❌

**Performance:**
- ✅ 2-3x mais rápido que Go
- ⚠️ Mas Go já é rápido o suficiente

**Complexidade:**
- ❌ Curva de aprendizado íngreme (6+ meses)
- ❌ Borrow checker é difícil
- ❌ Compile time lento

**Desenvolvimento:**
- ❌ Código mais verboso
- ❌ Menos produtivo
- ❌ Debugging mais difícil

**Time:**
- ❌ Difícil contratar devs Rust
- ❌ Onboarding longo (3-6 meses)
- ❌ Custo de treinamento alto

**Quando Rust faria sentido:**
- Se performance fosse crítica (< 1ms)
- Se tivéssemos time experiente em Rust
- Se fosse sistema embarcado/real-time

**Conclusão:**
Go oferece **80% da performance de Rust** com **20% da complexidade**.

## Consequências

### Positivas

✅ **Tecnologias Otimizadas:**
- Go para combates (10x mais rápido que Node.js, concorrência nativa)
- Python para IA (melhores libs)
- Node.js para tempo real (WebSocket)

✅ **Escalabilidade Independente:**
```
Dice Roller: 10 instâncias ($100/mês)
Story Generator: 1 instância ($20/mês)
vs Monolito: 10 instâncias de tudo ($1000/mês)
```

✅ **Controle de Custos:**
```
Free tier: Desliga serviços de IA
Pro tier: Rate limiting moderado
Master tier: Rate limiting alto
```

✅ **Fail Gracefully:**
- Story Generator cai → Resto funciona
- Battle Simulator cai → Personagens continuam funcionando

✅ **Deploy Independente:**
- Atualizar Dice Roller não afeta Character Generator
- Rollback isolado

✅ **Times Independentes:**
- Time de IA trabalha em Python
- Time de performance trabalha em Rust
- Sem conflitos

### Negativas

⚠️ **Complexidade Operacional:**
- 8 serviços para gerenciar
- Monitoramento distribuído
- Debugging mais difícil

**Mitigação:**
- Docker Compose para dev local
- Kubernetes para produção
- Observabilidade desde o início (logs, métricas, tracing)

⚠️ **Overhead de Rede:**
- Latência entre serviços
- Serialização/deserialização

**Mitigação:**
- Comunicação assíncrona (Kafka)
- Cache agressivo (Redis)
- gRPC para comunicação interna (futuro)

⚠️ **Consistência Eventual:**
- Dados podem estar desatualizados entre serviços

**Mitigação:**
- Aceitável para nosso domínio
- Analytics pode ter delay de 1-2s
- Operações críticas são síncronas

⚠️ **Custo Inicial:**
- Mais infraestrutura
- Mais ferramentas (Kafka, Redis, etc)

**Mitigação:**
- Justificado pelo modelo de negócio
- Economia a longo prazo

## Métricas de Sucesso

| Métrica | Baseline | Target | Prazo |
|---------|----------|--------|-------|
| Latência P95 (API Gateway) | - | < 200ms | 3 meses |
| Latência P95 (Character Gen) | - | < 5s | 3 meses |
| Throughput (Dice Roller) | - | > 1000 req/s | 6 meses |
| Uptime por serviço | - | > 99.9% | 6 meses |
| Custo por usuário Free | - | < $0.10/mês | 3 meses |
| Custo por usuário Pro | - | < $5/mês | 3 meses |
| Deploy time | - | < 10min | 3 meses |
| MTTR (Mean Time To Recovery) | - | < 15min | 6 meses |

## Riscos e Mitigações

| Risco | Probabilidade | Impacto | Mitigação |
|-------|---------------|---------|-----------|
| Complexidade alta | Alta | Médio | Documentação detalhada, treinamento |
| Debugging difícil | Média | Alto | Distributed tracing (Jaeger), logs estruturados |
| Custos de infra | Baixa | Alto | Monitoramento de custos, auto-scaling |
| Latência entre serviços | Média | Médio | Cache, comunicação assíncrona |
| Consistência de dados | Baixa | Médio | Event sourcing, saga pattern |

## Roadmap de Implementação

### Fase 1: MVP (3 meses)
1. API Gateway (autenticação básica)
2. Character Generator (sem IA)
3. Dice Roller (básico)
4. PostgreSQL único (compartilhado)
5. Kafka básico

### Fase 2: Microserviços (3 meses)
1. Separar databases
2. Adicionar Battle Simulator (Rust)
3. Adicionar Story Generator (Python)
4. Kafka completo

### Fase 3: IA (2 meses)
1. Integrar OpenAI no Character Generator
2. Integrar DALL-E
3. Narrativa de combates

### Fase 4: Escalabilidade (2 meses)
1. Kubernetes
2. Auto-scaling
3. Redis cache
4. Analytics Service

## Referências

- [Microservices Patterns - Chris Richardson](https://microservices.io/patterns/index.html)
- [Building Microservices - Sam Newman](https://samnewman.io/books/building_microservices/)
- [Domain-Driven Design - Eric Evans](https://www.domainlanguage.com/ddd/)
- [The Twelve-Factor App](https://12factor.net/)

## Histórico de Revisões

| Data | Versão | Autor | Descrição |
|------|--------|-------|-----------|
| 2025-10-05 | 1.0 | Laura Fumagalli | Decisão inicial de arquitetura de microserviços |

## Aprovações

| Papel | Nome | Data | Status |
|-------|------|------|--------|
| Arquiteto | Laura Fumagalli | 2025-10-05 | ✅ Aprovado |
| Tech Lead | - | - | Pendente |
| Product Owner | - | - | Pendente |

---

**Documentos Relacionados:**
- [Arquitetura Completa](../ARCHITECTURE.md)
- [Modelo de Negócio](../BUSINESS_MODEL.md)
- [ADR 002: Estratégia de Dados](./002-estrategia-dados.md)
- [ADR 003: Integração com IA](./003-integracao-ia.md)
