# 🏗️ Arquitetura do RPG System

> **Visão completa da arquitetura de microserviços, event-driven e integração com IA**

## 📋 Índice

- [Visão Geral](#visão-geral)
- [Princípios Arquiteturais](#princípios-arquiteturais)
- [Microserviços](#microserviços)
- [Comunicação](#comunicação)
- [Estratégia de Dados](#estratégia-de-dados)
- [Inteligência Artificial](#inteligência-artificial)
- [Escalabilidade](#escalabilidade)
- [Segurança](#segurança)

---

## Visão Geral

O RPG System é construído com uma arquitetura de **microserviços event-driven**, onde cada serviço é independente, escalável e focado em uma responsabilidade específica.

### Diagrama de Alto Nível

```
┌─────────────────────────────────────────────────────────────┐
│                        Clients                               │
│  Web App (React) │ Mobile App │ API Consumers               │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ↓
┌─────────────────────────────────────────────────────────────┐
│                    API Gateway (NestJS)                      │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐        │
│  │     Auth     │ │ Rate Limiting│ │   Routing    │        │
│  └──────────────┘ └──────────────┘ └──────────────┘        │
└────┬────────┬────────┬────────┬────────┬────────┬──────────┘
     │        │        │        │        │        │
     ↓        ↓        ↓        ↓        ↓        ↓
┌─────────┐ ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐ ┌──────────┐
│Character│ │Battle│ │Story │ │Dice  │ │Notify│ │Analytics │
│Generator│ │Sim   │ │Gen   │ │Roller│ │      │ │          │
│         │ │      │ │      │ │      │ │      │ │          │
│NestJS   │ │Go    │ │Python│ │Node  │ │Node  │ │Python    │
│OpenAI   │ │      │ │OpenAI│ │      │ │      │ │Pandas    │
│DALL-E   │ │      │ │      │ │      │ │      │ │          │
└────┬────┘ └──┬───┘ └──┬───┘ └──┬───┘ └──┬───┘ └────┬─────┘
     │         │        │        │        │          │
     │         │        │        │        │          │
     └─────────┴────────┴────────┴────────┴──────────┘
                         │
                         ↓
┌─────────────────────────────────────────────────────────────┐
│                    Event Bus (Kafka)                         │
│  Topics:                                                     │
│  • character.created                                        │
│  • character.updated                                        │
│  • battle.started                                           │
│  • battle.ended                                             │
│  • story.generated                                          │
│  • dice.rolled                                              │
│  • campaign.session.scheduled                               │
└─────────────────────────────────────────────────────────────┘
     │         │        │        │        │          │
     ↓         ↓        ↓        ↓        ↓          ↓
┌─────────┐ ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐ ┌──────────┐
│PostgreSQL│ │MongoDB│ │Postgres│Redis │ │SendGrid│TimescaleDB│
│         │ │       │ │       │      │ │        │          │
│Characters│ │Battles│ │Stories│Dice  │ │Email   │Metrics   │
│Campaigns │ │       │ │NPCs   │Cache │ │Push    │Events    │
└─────────┘ └──────┘ └──────┘ └──────┘ └──────┘ └──────────┘
```

---

## Princípios Arquiteturais

### 1. **Single Responsibility**
Cada microserviço tem uma responsabilidade única e bem definida.

### 2. **Database per Service**
Cada serviço possui seu próprio banco de dados, evitando acoplamento.

### 3. **Event-Driven Communication**
Comunicação assíncrona via Kafka para desacoplamento temporal.

### 4. **API Gateway Pattern**
Ponto único de entrada para todos os clientes.

### 5. **Fail Fast & Graceful Degradation**
Serviços falham rapidamente e o sistema continua funcionando parcialmente.

### 6. **Observability First**
Logs estruturados, métricas e tracing distribuído desde o início.

---

## Microserviços

### 1. API Gateway

**Responsabilidade:** Ponto único de entrada, autenticação e roteamento

**Tech Stack:**
- NestJS (Node.js)
- JWT para autenticação
- Redis para rate limiting

**Endpoints:**
```
POST   /auth/login
POST   /auth/register
GET    /characters
POST   /characters/generate
POST   /battles/simulate
POST   /stories/generate
POST   /dice/roll
GET    /campaigns
```

**Por que é necessário:**
- Centraliza autenticação
- Simplifica cliente (um único endpoint)
- Rate limiting centralizado
- CORS e segurança

---

### 2. Character Generator Service

**Responsabilidade:** Gerar personagens com IA

**Tech Stack:**
- NestJS (Node.js)
- OpenAI GPT-4 API
- DALL-E 3 API
- PostgreSQL

**Fluxo:**
```
1. Recebe descrição em linguagem natural
2. Usa GPT-4 para interpretar e gerar stats
3. Usa DALL-E para gerar avatar
4. Salva no PostgreSQL
5. Publica evento "character.created"
```

**Por que é um microserviço:**
- Processamento pesado (10-30 segundos)
- Usa APIs externas caras (rate limiting específico)
- Pode escalar independentemente
- Falha não afeta outros serviços

**Database Schema:**
```sql
CREATE TABLE characters (
  id UUID PRIMARY KEY,
  user_id UUID NOT NULL,
  system VARCHAR NOT NULL,  -- dnd-5e, vampire-v20
  name VARCHAR NOT NULL,
  description TEXT,
  stats JSONB NOT NULL,
  backstory TEXT,
  avatar_url VARCHAR,
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_characters_user ON characters(user_id);
CREATE INDEX idx_characters_system ON characters(system);
```

---

### 3. Battle Simulator Service

**Responsabilidade:** Simular combates entre personagens

**Tech Stack:**
- Go (performance e concorrência)
- MongoDB (armazenar histórico de combates)
- OpenAI (narrativa)

**Fluxo:**
```
1. Recebe dois personagens
2. Simula combate round por round (Go - rápido e concorrente)
3. Usa GPT-4 para narrar o combate
4. Salva resultado no MongoDB
5. Publica evento "battle.ended"
```

**Por que é um microserviço:**
- CPU-intensive (milhares de cálculos)
- Go oferece excelente performance e concorrência (goroutines)
- Pode escalar horizontalmente facilmente
- Domínio completamente diferente
- Deployment simples (binário único)

**Por que Go (não Rust):**
- ✅ Mais simples de aprender e manter
- ✅ Goroutines perfeitas para simular múltiplos combates
- ✅ Performance suficiente (10x mais rápido que Node.js)
- ✅ Ecossistema melhor para microserviços
- ✅ Mais fácil contratar desenvolvedores Go

**Por que MongoDB:**
- Combates são documentos (não relacionais)
- Schema flexível (diferentes tipos de combate)
- Alta performance em writes

**Document Schema:**
```json
{
  "_id": "uuid",
  "character1_id": "uuid",
  "character2_id": "uuid",
  "rounds": [
    {
      "number": 1,
      "attacker": "character1",
      "action": "attack",
      "roll": 18,
      "damage": 12,
      "narrative": "Grog swings his axe..."
    }
  ],
  "winner": "character1",
  "duration_ms": 1234,
  "created_at": "2025-01-01T00:00:00Z"
}
```

---

### 4. Story Generator Service

**Responsabilidade:** Gerar histórias, NPCs e missões

**Tech Stack:**
- Python (melhor para ML/IA)
- OpenAI GPT-4 API
- PostgreSQL

**Fluxo:**
```
1. Recebe contexto (campanha, nível dos jogadores)
2. Usa GPT-4 para gerar história/NPC/missão
3. Salva no PostgreSQL
4. Publica evento "story.generated"
```

**Por que é um microserviço:**
- Usa LLM (caro, precisa rate limiting)
- Python tem melhores libs de IA
- Pode ser desligado em free tier
- Domínio específico (geração de conteúdo)

**Database Schema:**
```sql
CREATE TABLE stories (
  id UUID PRIMARY KEY,
  campaign_id UUID,
  type VARCHAR NOT NULL,  -- quest, npc, location
  title VARCHAR NOT NULL,
  content TEXT NOT NULL,
  metadata JSONB,
  created_at TIMESTAMP DEFAULT NOW()
);
```

---

### 5. Dice Roller Service

**Responsabilidade:** Rolagem de dados em tempo real

**Tech Stack:**
- Node.js (WebSocket)
- Redis (cache e histórico)

**Fluxo:**
```
1. Recebe pedido de rolagem (1d20+5)
2. Calcula resultado
3. Salva no Redis (histórico)
4. Analisa padrões (detecção de trapaça)
5. Publica evento "dice.rolled"
6. Envia via WebSocket (tempo real)
```

**Por que é um microserviço:**
- Tempo real (WebSocket)
- Alta frequência (milhares de rolagens/segundo)
- Precisa baixa latência
- Pode escalar horizontalmente

**Por que Redis:**
- Extremamente rápido (in-memory)
- TTL automático para histórico
- Pub/Sub nativo para WebSocket

**Redis Structure:**
```
dice:history:{user_id} -> LIST de rolagens (últimas 100)
dice:stats:{user_id} -> HASH com estatísticas
```

---

### 6. Notification Service

**Responsabilidade:** Enviar notificações (email, push, WebSocket)

**Tech Stack:**
- Node.js
- SendGrid (email)
- Firebase (push notifications)
- WebSocket

**Fluxo:**
```
1. Consome eventos do Kafka
2. Decide qual notificação enviar
3. Envia via canal apropriado
4. Registra entrega
```

**Por que é um microserviço:**
- Pode falhar sem afetar outros serviços
- Infraestrutura externa (SendGrid, Firebase)
- Rate limiting específico
- Pode ser substituído facilmente

---

### 7. Analytics Service

**Responsabilidade:** Coletar e analisar métricas

**Tech Stack:**
- Python (Pandas, NumPy)
- TimescaleDB (séries temporais)

**Fluxo:**
```
1. Consome todos os eventos do Kafka
2. Agrega métricas
3. Calcula insights
4. Armazena no TimescaleDB
5. Expõe API de consulta
```

**Por que é um microserviço:**
- Processamento batch (não afeta tempo real)
- Big data (milhões de eventos)
- Python tem melhores libs de análise
- Pode ser desligado sem afetar sistema

**Por que TimescaleDB:**
- Otimizado para séries temporais
- Queries agregadas rápidas
- Retenção automática de dados

---

### 8. Campaign Manager Service

**Responsabilidade:** Gerenciar campanhas e sessões

**Tech Stack:**
- NestJS (Node.js)
- PostgreSQL

**Database Schema:**
```sql
CREATE TABLE campaigns (
  id UUID PRIMARY KEY,
  master_id UUID NOT NULL,
  name VARCHAR NOT NULL,
  description TEXT,
  system VARCHAR NOT NULL,
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE campaign_players (
  campaign_id UUID REFERENCES campaigns(id),
  player_id UUID NOT NULL,
  character_id UUID,
  joined_at TIMESTAMP DEFAULT NOW(),
  PRIMARY KEY (campaign_id, player_id)
);

CREATE TABLE sessions (
  id UUID PRIMARY KEY,
  campaign_id UUID REFERENCES campaigns(id),
  scheduled_at TIMESTAMP NOT NULL,
  duration_minutes INT,
  notes TEXT,
  created_at TIMESTAMP DEFAULT NOW()
);
```

---

## Comunicação

### Síncrona (REST)

**Quando usar:**
- Cliente precisa resposta imediata
- Operações CRUD simples
- Queries

**Exemplo:**
```
GET /characters/{id} -> Character Generator
```

### Assíncrona (Kafka)

**Quando usar:**
- Operações longas (> 1s)
- Múltiplos consumidores
- Desacoplamento temporal

**Eventos:**

```typescript
// character.created
{
  eventType: 'character.created',
  aggregateId: 'uuid',
  timestamp: '2025-01-01T00:00:00Z',
  data: {
    characterId: 'uuid',
    userId: 'uuid',
    name: 'Grog',
    system: 'dnd-5e'
  }
}

// battle.ended
{
  eventType: 'battle.ended',
  aggregateId: 'uuid',
  timestamp: '2025-01-01T00:00:00Z',
  data: {
    battleId: 'uuid',
    winner: 'character1_id',
    duration: 1234
  }
}
```

---

## Estratégia de Dados

### Database per Service

Cada serviço tem seu próprio banco, escolhido para seu caso de uso:

| Serviço | Database | Por quê? |
|---------|----------|----------|
| Character Generator | PostgreSQL | Dados relacionais, transações |
| Battle Simulator | MongoDB | Documentos, schema flexível |
| Story Generator | PostgreSQL | Busca full-text, relacionamentos |
| Dice Roller | Redis | In-memory, alta performance |
| Analytics | TimescaleDB | Séries temporais |
| Campaign Manager | PostgreSQL | Relacionamentos complexos |

### Consistência Eventual

Aceitamos **eventual consistency** entre serviços:

```
1. Character Generator cria personagem
2. Publica evento "character.created"
3. Analytics consome evento (pode demorar 1-2s)
4. Usuário vê personagem imediatamente
5. Analytics atualiza estatísticas depois
```

---

## Inteligência Artificial

### OpenAI GPT-4

**Usado em:**
- Character Generator (interpretar descrição)
- Battle Simulator (narrar combate)
- Story Generator (gerar histórias)

**Rate Limiting:**
```typescript
// Implementado em cada serviço
const rateLimiter = new RateLimiter({
  maxRequests: 100,  // por minuto
  tier: user.subscriptionTier  // free, pro, master
});
```

### DALL-E 3

**Usado em:**
- Character Generator (gerar avatares)

**Otimizações:**
- Cache de avatares similares
- Geração assíncrona (não bloqueia)

---

## Escalabilidade

### Horizontal Scaling

Todos os serviços são **stateless** e podem escalar horizontalmente:

```yaml
# Kubernetes
apiVersion: apps/v1
kind: Deployment
metadata:
  name: character-generator
spec:
  replicas: 3  # Escala para 3 instâncias
  selector:
    matchLabels:
      app: character-generator
```

### Auto-scaling

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: character-generator-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: character-generator
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

### Caching Strategy

```
┌─────────┐
│ Client  │
└────┬────┘
     │
     ↓
┌─────────────┐
│ API Gateway │
│   + Redis   │  ← Cache de respostas (TTL: 5min)
└────┬────────┘
     │
     ↓
┌──────────────────┐
│ Character Service│
│    + Redis       │  ← Cache de personagens (TTL: 1h)
└────┬─────────────┘
     │
     ↓
┌──────────────┐
│ PostgreSQL   │
└──────────────┘
```

---

## Segurança

### Autenticação

```
JWT Token:
{
  "sub": "user_id",
  "email": "user@example.com",
  "tier": "pro",
  "iat": 1234567890,
  "exp": 1234571490
}
```

### Autorização

```typescript
// Guard no API Gateway
@UseGuards(JwtAuthGuard, SubscriptionGuard)
@RequiresTier('pro')
@Post('characters/generate-with-ai')
async generateWithAI() {
  // Só usuários Pro+ podem usar IA
}
```

### Rate Limiting

```typescript
// Por tier
const limits = {
  free: { requests: 10, window: '1h' },
  pro: { requests: 100, window: '1h' },
  master: { requests: 1000, window: '1h' }
};
```

---

## Observabilidade

### Logs Estruturados

```json
{
  "timestamp": "2025-01-01T00:00:00Z",
  "level": "info",
  "service": "character-generator",
  "correlationId": "uuid",
  "userId": "uuid",
  "message": "Character generated successfully",
  "duration": 1234,
  "metadata": {
    "characterId": "uuid",
    "system": "dnd-5e"
  }
}
```

### Métricas (Prometheus)

```
# Latência
http_request_duration_seconds{service="character-generator",endpoint="/generate"}

# Taxa de erro
http_requests_total{service="character-generator",status="500"}

# Uso de IA
openai_requests_total{service="character-generator",model="gpt-4"}
```

### Distributed Tracing (Jaeger)

```
Request ID: abc123
├─ API Gateway (10ms)
├─ Character Generator (5000ms)
│  ├─ OpenAI GPT-4 (4500ms)
│  ├─ DALL-E (3000ms)
│  └─ PostgreSQL (50ms)
└─ Kafka Publish (5ms)
```

---

## Próximos Passos

1. Implementar API Gateway
2. Implementar Character Generator (MVP sem IA)
3. Adicionar Kafka
4. Integrar OpenAI
5. Implementar outros microserviços

---

**Documentos Relacionados:**
- [ADR 001: Arquitetura de Microserviços](./adr/001-arquitetura-microservicos.md)
- [ADR 002: Estratégia de Dados](./adr/002-estrategia-dados.md)
- [ADR 003: Integração com IA](./adr/003-integracao-ia.md)
