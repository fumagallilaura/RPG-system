# ADR 002: Estratégia de Dados - Database per Service

- **Status:** Aceito
- **Data:** 2025-10-05
- **Autor:** Laura Fumagalli
- **Decisores:** Equipe de Arquitetura

## Contexto

Em uma arquitetura de microserviços, a estratégia de dados é crítica. Cada serviço tem necessidades diferentes:

### Requisitos por Serviço

| Serviço | Tipo de Dados | Padrão de Acesso | Volume |
|---------|---------------|------------------|--------|
| Character Generator | Relacional | CRUD + Queries complexas | Médio |
| Battle Simulator | Documentos | Write-heavy, queries simples | Alto |
| Story Generator | Relacional | CRUD + Full-text search | Baixo |
| Dice Roller | Cache/Histórico | Read/Write extremamente rápido | Muito Alto |
| Analytics | Séries Temporais | Agregações, time-series | Muito Alto |
| Campaign Manager | Relacional | CRUD + Relacionamentos | Médio |

### Desafios

1. **Consistência:** Como manter dados consistentes entre serviços?
2. **Queries Cross-Service:** Como buscar dados de múltiplos serviços?
3. **Transações:** Como garantir atomicidade entre serviços?
4. **Performance:** Como evitar N+1 queries?
5. **Custos:** Como otimizar custos de infraestrutura?

## Decisão

Adotar **Database per Service** com bancos de dados especializados para cada caso de uso.

### Mapeamento Serviço → Database

```
┌──────────────────────┐
│ Character Generator  │ → PostgreSQL
│ - Characters         │   (Relacional, ACID)
│ - Systems            │
│ - Templates          │
└──────────────────────┘

┌──────────────────────┐
│ Battle Simulator     │ → MongoDB
│ - Battles            │   (Documentos, Schema flexível)
│ - Combat Logs        │
└──────────────────────┘

┌──────────────────────┐
│ Story Generator      │ → PostgreSQL
│ - Stories            │   (Full-text search)
│ - NPCs               │
│ - Quests             │
└──────────────────────┘

┌──────────────────────┐
│ Dice Roller          │ → Redis
│ - Roll History       │   (In-memory, TTL)
│ - User Stats         │
└──────────────────────┘

┌──────────────────────┐
│ Analytics            │ → TimescaleDB
│ - Events             │   (Séries temporais)
│ - Metrics            │
└──────────────────────┘

┌──────────────────────┐
│ Campaign Manager     │ → PostgreSQL
│ - Campaigns          │   (Relacionamentos)
│ - Sessions           │
│ - Players            │
└──────────────────────┘
```

### Schemas Detalhados

#### 1. Character Generator (PostgreSQL)

```sql
-- Sistemas de RPG (D&D, Vampiro, etc)
CREATE TABLE systems (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(100) NOT NULL,
  version VARCHAR(50),
  rules JSONB NOT NULL,  -- Regras de geração
  created_at TIMESTAMP DEFAULT NOW()
);

-- Personagens
CREATE TABLE characters (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL,
  system_id UUID REFERENCES systems(id),
  name VARCHAR(100) NOT NULL,
  description TEXT,
  stats JSONB NOT NULL,  -- Atributos do personagem
  backstory TEXT,
  avatar_url VARCHAR(500),
  tier VARCHAR(20) NOT NULL,  -- free, pro, master
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_characters_user ON characters(user_id);
CREATE INDEX idx_characters_system ON characters(system_id);
CREATE INDEX idx_characters_tier ON characters(tier);

-- Histórico de gerações (para analytics)
CREATE TABLE generation_history (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  character_id UUID REFERENCES characters(id),
  prompt TEXT,  -- Descrição do usuário
  ai_cost DECIMAL(10, 4),  -- Custo da geração
  duration_ms INT,
  created_at TIMESTAMP DEFAULT NOW()
);
```

**Por que PostgreSQL:**
- ✅ ACID para criação de personagens
- ✅ Foreign Keys garantem integridade
- ✅ JSONB para stats flexíveis
- ✅ Índices eficientes

#### 2. Battle Simulator (MongoDB)

```javascript
// Collection: battles
{
  _id: ObjectId("..."),
  character1: {
    id: "uuid",
    name: "Grog",
    stats: { strength: 18, dexterity: 14, ... }
  },
  character2: {
    id: "uuid",
    name: "Goblin",
    stats: { strength: 8, dexterity: 16, ... }
  },
  rounds: [
    {
      number: 1,
      attacker: "character1",
      action: "attack",
      roll: 18,
      damage: 12,
      narrative: "Grog swings his axe with tremendous force...",
      timestamp: ISODate("2025-01-01T00:00:00Z")
    },
    {
      number: 2,
      attacker: "character2",
      action: "dodge",
      roll: 15,
      success: true,
      narrative: "The goblin nimbly dodges...",
      timestamp: ISODate("2025-01-01T00:00:01Z")
    }
  ],
  winner: "character1",
  duration_ms: 1234,
  ai_narrative: "In an epic battle...",
  metadata: {
    system: "dnd-5e",
    difficulty: "medium",
    user_id: "uuid"
  },
  created_at: ISODate("2025-01-01T00:00:00Z")
}

// Índices
db.battles.createIndex({ "metadata.user_id": 1, "created_at": -1 });
db.battles.createIndex({ "winner": 1 });
db.battles.createIndex({ "metadata.system": 1 });
```

**Por que MongoDB:**
- ✅ Combates são documentos auto-contidos
- ✅ Schema flexível (diferentes tipos de combate)
- ✅ Alta performance em writes
- ✅ Fácil de escalar horizontalmente
- ✅ Queries simples (não precisa JOINs)

#### 3. Story Generator (PostgreSQL)

```sql
-- Histórias, NPCs, Missões
CREATE TABLE stories (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  campaign_id UUID,  -- Opcional, pode ser genérico
  user_id UUID NOT NULL,
  type VARCHAR(50) NOT NULL,  -- quest, npc, location, plot
  title VARCHAR(200) NOT NULL,
  content TEXT NOT NULL,
  metadata JSONB,  -- Tags, dificuldade, etc
  ai_prompt TEXT,  -- Prompt usado
  ai_cost DECIMAL(10, 4),
  search_vector tsvector,  -- Full-text search
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_stories_user ON stories(user_id);
CREATE INDEX idx_stories_campaign ON stories(campaign_id);
CREATE INDEX idx_stories_type ON stories(type);
CREATE INDEX idx_stories_search ON stories USING GIN(search_vector);

-- Trigger para atualizar search_vector
CREATE TRIGGER stories_search_update
BEFORE INSERT OR UPDATE ON stories
FOR EACH ROW EXECUTE FUNCTION
  tsvector_update_trigger(search_vector, 'pg_catalog.english', title, content);
```

**Por que PostgreSQL:**
- ✅ Full-text search nativo
- ✅ Relacionamento com campanhas
- ✅ JSONB para metadata flexível
- ✅ Transações para operações complexas

#### 4. Dice Roller (Redis)

```redis
# Histórico de rolagens (lista com TTL de 24h)
LPUSH dice:history:{user_id} "{\"dice\":\"1d20+5\",\"result\":23,\"timestamp\":1234567890}"
LTRIM dice:history:{user_id} 0 99  # Mantém últimas 100
EXPIRE dice:history:{user_id} 86400  # 24 horas

# Estatísticas do usuário (hash)
HSET dice:stats:{user_id} total_rolls 1234
HSET dice:stats:{user_id} critical_hits 45
HSET dice:stats:{user_id} critical_fails 38
HSET dice:stats:{user_id} average_roll 11.2

# Detecção de trapaça (sorted set)
ZADD dice:suspicion {score} {user_id}  # Score baseado em análise estatística

# Cache de sessão (string com TTL)
SET dice:session:{session_id} "{\"user_id\":\"...\",\"campaign_id\":\"...\"}" EX 3600
```

**Por que Redis:**
- ✅ Extremamente rápido (in-memory)
- ✅ TTL automático para histórico
- ✅ Pub/Sub para WebSocket
- ✅ Estruturas de dados especializadas
- ✅ Baixa latência (< 1ms)

#### 5. Analytics (TimescaleDB)

```sql
-- Hypertable para eventos
CREATE TABLE events (
  time TIMESTAMPTZ NOT NULL,
  event_type VARCHAR(50) NOT NULL,
  user_id UUID,
  service VARCHAR(50) NOT NULL,
  metadata JSONB,
  duration_ms INT,
  success BOOLEAN
);

-- Converte para hypertable (TimescaleDB)
SELECT create_hypertable('events', 'time');

-- Índices
CREATE INDEX idx_events_type ON events(event_type, time DESC);
CREATE INDEX idx_events_user ON events(user_id, time DESC);
CREATE INDEX idx_events_service ON events(service, time DESC);

-- Políticas de retenção
SELECT add_retention_policy('events', INTERVAL '90 days');

-- Continuous aggregates (pré-computadas)
CREATE MATERIALIZED VIEW events_hourly
WITH (timescaledb.continuous) AS
SELECT
  time_bucket('1 hour', time) AS hour,
  event_type,
  service,
  COUNT(*) as count,
  AVG(duration_ms) as avg_duration,
  SUM(CASE WHEN success THEN 1 ELSE 0 END) as success_count
FROM events
GROUP BY hour, event_type, service;
```

**Por que TimescaleDB:**
- ✅ Otimizado para séries temporais
- ✅ Queries agregadas extremamente rápidas
- ✅ Retenção automática de dados
- ✅ Continuous aggregates (pré-computação)
- ✅ Compatível com PostgreSQL

#### 6. Campaign Manager (PostgreSQL)

```sql
-- Campanhas
CREATE TABLE campaigns (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  master_id UUID NOT NULL,
  name VARCHAR(200) NOT NULL,
  description TEXT,
  system VARCHAR(50) NOT NULL,  -- dnd-5e, vampire-v20
  is_public BOOLEAN DEFAULT false,
  max_players INT DEFAULT 6,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

-- Jogadores na campanha
CREATE TABLE campaign_players (
  campaign_id UUID REFERENCES campaigns(id) ON DELETE CASCADE,
  player_id UUID NOT NULL,
  character_id UUID,  -- Referência lógica (não FK)
  role VARCHAR(20) DEFAULT 'player',  -- player, co-master
  joined_at TIMESTAMP DEFAULT NOW(),
  PRIMARY KEY (campaign_id, player_id)
);

-- Sessões
CREATE TABLE sessions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  campaign_id UUID REFERENCES campaigns(id) ON DELETE CASCADE,
  scheduled_at TIMESTAMP NOT NULL,
  duration_minutes INT,
  notes TEXT,
  summary TEXT,  -- Gerado por IA
  created_at TIMESTAMP DEFAULT NOW()
);

-- Log de eventos da campanha
CREATE TABLE campaign_events (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  campaign_id UUID REFERENCES campaigns(id) ON DELETE CASCADE,
  session_id UUID REFERENCES sessions(id) ON DELETE SET NULL,
  event_type VARCHAR(50) NOT NULL,  -- combat, story, loot
  description TEXT NOT NULL,
  metadata JSONB,
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_campaigns_master ON campaigns(master_id);
CREATE INDEX idx_campaign_players_player ON campaign_players(player_id);
CREATE INDEX idx_sessions_campaign ON sessions(campaign_id, scheduled_at);
CREATE INDEX idx_events_campaign ON campaign_events(campaign_id, created_at DESC);
```

**Por que PostgreSQL:**
- ✅ Relacionamentos complexos (campanhas, jogadores, sessões)
- ✅ Foreign Keys garantem integridade
- ✅ Transações para operações multi-tabela
- ✅ Queries complexas com JOINs

### Comunicação Entre Serviços

#### Problema: Queries Cross-Service

**Exemplo:** Buscar todos os combates de um personagem

```
Character Generator tem: character_id
Battle Simulator tem: battles com character1_id, character2_id

Como buscar?
```

**Solução 1: API Composition (Síncrona)**

```typescript
// No API Gateway
async getCharacterWithBattles(characterId: string) {
  // 1. Busca personagem
  const character = await characterService.getCharacter(characterId);
  
  // 2. Busca combates (paralelo)
  const battles = await battleService.getBattlesByCharacter(characterId);
  
  // 3. Combina
  return {
    ...character,
    battles
  };
}
```

**Vantagens:**
- ✅ Simples
- ✅ Dados sempre atualizados

**Desvantagens:**
- ⚠️ Latência (múltiplas chamadas)
- ⚠️ Acoplamento temporal

**Solução 2: Event-Driven (Assíncrona)**

```typescript
// Battle Simulator publica evento
await kafka.publish('battle.ended', {
  battleId: 'uuid',
  character1Id: 'uuid',
  character2Id: 'uuid',
  winner: 'character1Id'
});

// Analytics consome e agrega
@EventPattern('battle.ended')
async handleBattleEnded(data: BattleEndedEvent) {
  // Salva estatísticas agregadas
  await this.analyticsRepo.incrementBattleCount(data.character1Id);
  await this.analyticsRepo.updateWinRate(data.winner);
}
```

**Vantagens:**
- ✅ Desacoplamento
- ✅ Escalável
- ✅ Eventual consistency aceitável

**Desvantagens:**
- ⚠️ Dados podem estar desatualizados (1-2s)

#### Problema: Transações Distribuídas

**Exemplo:** Criar campanha e adicionar jogadores

```
Campaign Manager: INSERT campaign
Campaign Manager: INSERT campaign_players
```

**Solução: Saga Pattern**

```typescript
// Saga: CreateCampaignSaga
async execute(command: CreateCampaignCommand) {
  const saga = new Saga();
  
  try {
    // Step 1: Criar campanha
    const campaign = await saga.step(
      () => this.campaignRepo.create(command),
      (campaign) => this.campaignRepo.delete(campaign.id) // Compensação
    );
    
    // Step 2: Adicionar jogadores
    await saga.step(
      () => this.campaignRepo.addPlayers(campaign.id, command.players),
      () => this.campaignRepo.removePlayers(campaign.id, command.players)
    );
    
    // Step 3: Publicar evento
    await saga.step(
      () => this.kafka.publish('campaign.created', campaign),
      () => this.kafka.publish('campaign.creation.failed', campaign)
    );
    
    await saga.commit();
    return campaign;
    
  } catch (error) {
    await saga.rollback();  // Executa compensações
    throw error;
  }
}
```

### Consistência Eventual

Aceitamos **eventual consistency** entre serviços:

```
1. Character Generator cria personagem
   └─ Publica: character.created

2. Analytics consome evento (delay: 100ms-2s)
   └─ Atualiza: total_characters++

3. Usuário vê personagem imediatamente ✅
4. Analytics atualiza depois ✅ (aceitável)
```

**Quando eventual consistency NÃO é aceitável:**
- Pagamentos
- Autenticação
- Operações críticas de negócio

**Nesses casos:** Usamos comunicação síncrona

## Alternativas Consideradas

### Alternativa 1: Shared Database

**Estrutura:**
```
Todos os serviços → PostgreSQL único
```

**Vantagens:**
- ✅ Transações ACID entre serviços
- ✅ Queries cross-service simples
- ✅ Consistência forte

**Desvantagens:**
- ❌ Acoplamento forte (schema compartilhado)
- ❌ Não pode usar bancos especializados
- ❌ Gargalo de performance
- ❌ Deploy acoplado (migrations)

**Por que rejeitamos:**
- MongoDB é muito melhor para combates
- Redis é essencial para dice roller
- TimescaleDB é otimizado para analytics

### Alternativa 2: Event Sourcing

**Estrutura:**
```
Todos os eventos → Event Store
Cada serviço reconstrói estado
```

**Vantagens:**
- ✅ Histórico completo
- ✅ Audit trail
- ✅ Time travel

**Desvantagens:**
- ❌ Complexidade muito alta
- ❌ Queries complexas
- ❌ Overkill para MVP

**Por que rejeitamos:**
- Complexidade desnecessária para início
- Pode adicionar depois se necessário

### Alternativa 3: CQRS (Command Query Responsibility Segregation)

**Estrutura:**
```
Write Model: PostgreSQL
Read Model: MongoDB (desnormalizado)
```

**Vantagens:**
- ✅ Otimizado para leitura
- ✅ Escalável

**Desvantagens:**
- ❌ Complexidade alta
- ❌ Sincronização entre modelos
- ❌ Overkill para MVP

**Por que rejeitamos:**
- Pode adicionar depois se necessário
- Database per service já resolve escalabilidade

## Consequências

### Positivas

✅ **Bancos Especializados:**
- MongoDB para documentos (combates)
- Redis para cache (dice roller)
- TimescaleDB para séries temporais (analytics)

✅ **Desacoplamento:**
- Cada serviço evolui independentemente
- Migrations isoladas
- Sem conflitos de schema

✅ **Escalabilidade:**
- Escala bancos independentemente
- Read replicas por serviço
- Sharding específico

✅ **Performance:**
- Banco otimizado para cada caso
- Sem queries cross-database lentas
- Cache agressivo possível

### Negativas

⚠️ **Consistência Eventual:**
- Dados podem estar desatualizados (1-2s)

**Mitigação:**
- Aceitável para nosso domínio
- Operações críticas são síncronas

⚠️ **Queries Cross-Service:**
- Mais complexas
- API Composition ou Event-Driven

**Mitigação:**
- API Gateway faz composição
- Cache para queries frequentes

⚠️ **Transações Distribuídas:**
- Saga Pattern é complexo

**Mitigação:**
- Minimizar transações cross-service
- Design cuidadoso de bounded contexts

⚠️ **Custo de Infraestrutura:**
- Múltiplos bancos

**Mitigação:**
- Justificado pela performance
- Managed services (RDS, Atlas, etc)

## Métricas de Sucesso

| Métrica | Target |
|---------|--------|
| Latência de queries (P95) | < 100ms |
| Latência cross-service (P95) | < 200ms |
| Uptime por database | > 99.9% |
| Custo por usuário (database) | < $0.50/mês |
| Tempo de migration | < 5min |
| Consistência eventual (delay) | < 2s |

## Referências

- [Database per Service Pattern](https://microservices.io/patterns/data/database-per-service.html)
- [Saga Pattern](https://microservices.io/patterns/data/saga.html)
- [Event Sourcing](https://martinfowler.com/eaaDev/EventSourcing.html)
- [CQRS](https://martinfowler.com/bliki/CQRS.html)

## Histórico de Revisões

| Data | Versão | Autor | Descrição |
|------|--------|-------|-----------|
| 2025-10-05 | 1.0 | Laura Fumagalli | Estratégia inicial de dados |

---

**Documentos Relacionados:**
- [ADR 001: Arquitetura de Microserviços](./001-arquitetura-microservicos.md)
- [ADR 003: Integração com IA](./003-integracao-ia.md)
- [Arquitetura Completa](../ARCHITECTURE.md)
