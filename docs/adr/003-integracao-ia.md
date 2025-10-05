# ADR 003: Integração com Inteligência Artificial

- **Status:** Aceito
- **Data:** 2025-10-05
- **Autor:** Laura Fumagalli
- **Decisores:** Equipe de Arquitetura, Product Owner

## Contexto

A IA é o **diferencial competitivo** do RPG System. Concorrentes oferecem ferramentas básicas, mas nenhum tem IA integrada para:

1. **Interpretar linguagem natural** → Gerar personagens
2. **Criar narrativas dinâmicas** → Combates interessantes
3. **Gerar conteúdo infinito** → Missões, NPCs, histórias
4. **Detectar padrões** → Anti-trapaça em rolagens

### Requisitos

1. **Qualidade:** IA precisa gerar conteúdo coerente e útil
2. **Custo:** IA é cara ($0.03-0.20 por geração)
3. **Latência:** Usuário não pode esperar > 30s
4. **Escalabilidade:** Precisa suportar milhares de usuários
5. **Monetização:** IA justifica tiers pagos

### Desafios

1. **Custos de OpenAI:**
   - GPT-4: $0.03/1K tokens (input), $0.06/1K tokens (output)
   - DALL-E 3: $0.04/imagem
   - Pode ficar caro rapidamente

2. **Rate Limiting:**
   - OpenAI tem limites por tier
   - Precisa controlar uso por usuário

3. **Latência:**
   - GPT-4 pode demorar 5-15s
   - DALL-E pode demorar 10-20s
   - Usuário precisa de feedback

4. **Qualidade:**
   - IA pode gerar conteúdo inadequado
   - Precisa validação e filtros

## Decisão

Integrar **OpenAI GPT-4** e **DALL-E 3** com estratégia de controle de custos e qualidade.

### Onde Usar IA

#### 1. Character Generator Service

**Funcionalidade:** Interpretar descrição e gerar personagem completo

**Input do Usuário:**
```
"Quero um mago elfo velho e sábio, especialista em magia de fogo.
Ele passou décadas estudando nas bibliotecas antigas."
```

**Prompt para GPT-4:**
```typescript
const prompt = `
Você é um especialista em RPG ${system} (ex: D&D 5e).

Baseado na descrição do usuário, gere um personagem completo em JSON:

Descrição: "${userDescription}"

Retorne JSON com:
{
  "name": "Nome apropriado",
  "race": "Raça adequada",
  "class": "Classe principal",
  "subclass": "Subclasse se aplicável",
  "level": número entre 1-20,
  "stats": {
    "strength": número,
    "dexterity": número,
    "constitution": número,
    "intelligence": número,
    "wisdom": número,
    "charisma": número
  },
  "skills": ["skill1", "skill2"],
  "spells": ["spell1", "spell2"],
  "equipment": ["item1", "item2"],
  "backstory": "História completa e coerente (200-300 palavras)",
  "personality": "Traços de personalidade",
  "ideals": "Ideais do personagem",
  "bonds": "Laços importantes",
  "flaws": "Falhas/fraquezas"
}

Regras:
- Stats devem somar entre 70-80 (padrão point-buy)
- Spells devem ser apropriados para classe e nível
- Backstory deve ser coerente com descrição
- Use nomes apropriados para raça/cultura
`;

const response = await openai.chat.completions.create({
  model: 'gpt-4-turbo-preview',
  messages: [{ role: 'user', content: prompt }],
  temperature: 0.7,  // Criatividade moderada
  max_tokens: 2000,
  response_format: { type: 'json_object' }
});
```

**Output:**
```json
{
  "name": "Eldrin Flameheart",
  "race": "High Elf",
  "class": "Wizard",
  "subclass": "School of Evocation",
  "level": 10,
  "stats": {
    "strength": 8,
    "dexterity": 14,
    "constitution": 12,
    "intelligence": 18,
    "wisdom": 16,
    "charisma": 10
  },
  "skills": ["Arcana", "History", "Investigation"],
  "spells": ["Fireball", "Wall of Fire", "Flame Strike"],
  "equipment": ["Staff of Fire", "Spellbook", "Component Pouch"],
  "backstory": "Eldrin passou 300 anos nas bibliotecas antigas...",
  "personality": "Sábio, paciente, mas impulsivo quando provocado",
  "ideals": "Conhecimento é poder",
  "bonds": "Proteger as bibliotecas antigas",
  "flaws": "Arrogante com magos menos experientes"
}
```

**Custo Estimado:**
- Input: ~500 tokens = $0.015
- Output: ~800 tokens = $0.048
- **Total: ~$0.06 por personagem**

**Avatar com DALL-E:**
```typescript
const avatarPrompt = `
Portrait of ${character.name}, a ${character.race} ${character.class}.
${character.description}
Fantasy art style, detailed, high quality.
`;

const image = await openai.images.generate({
  model: 'dall-e-3',
  prompt: avatarPrompt,
  size: '1024x1024',
  quality: 'standard'
});
```

**Custo:** $0.04 por imagem

**Total por personagem com IA: ~$0.10**

---

#### 2. Battle Simulator Service

**Funcionalidade:** Narrar combate de forma dinâmica

**Input:**
```json
{
  "character1": {
    "name": "Grog",
    "class": "Barbarian",
    "level": 10
  },
  "character2": {
    "name": "Goblin Warrior",
    "type": "monster",
    "cr": 1
  },
  "rounds": [
    {
      "attacker": "Grog",
      "action": "attack",
      "roll": 18,
      "damage": 24,
      "hit": true
    },
    {
      "attacker": "Goblin",
      "action": "attack",
      "roll": 8,
      "damage": 0,
      "hit": false
    },
    {
      "attacker": "Grog",
      "action": "attack",
      "roll": 20,
      "damage": 48,
      "hit": true,
      "critical": true
    }
  ],
  "winner": "Grog"
}
```

**Prompt para GPT-4:**
```typescript
const prompt = `
Você é um narrador de RPG experiente.

Narre o combate abaixo de forma épica e envolvente:

${JSON.stringify(battleData, null, 2)}

Regras:
- Máximo 500 palavras
- Descreva cada round com detalhes
- Use linguagem épica mas clara
- Mencione críticos e falhas de forma dramática
- Finalize com a vitória
`;

const narrative = await openai.chat.completions.create({
  model: 'gpt-4-turbo-preview',
  messages: [{ role: 'user', content: prompt }],
  temperature: 0.8,  // Mais criativo
  max_tokens: 800
});
```

**Output:**
```
"Grog, o poderoso bárbaro, avança com sua machado em punho, 
os músculos tensos sob a pele marcada por cicatrizes de batalhas 
passadas. O goblin guerreiro, menor mas ágil, tenta se esquivar, 
mas não é páreo para a fúria do bárbaro.

Com um golpe devastador, Grog acerta o goblin com força brutal 
(24 de dano!). O goblin cambaleante tenta contra-atacar, mas sua 
lâmina encontra apenas o ar.

Então, em um momento decisivo, Grog ergue seu machado alto e, 
com um grito de guerra que ecoa pelas montanhas, desfere um 
GOLPE CRÍTICO! (48 de dano!) O machado encontra seu alvo com 
precisão mortal, e o goblin cai derrotado.

Grog emerge vitorioso, respirando pesadamente, mas triunfante!"
```

**Custo:** ~$0.05 por combate

---

#### 3. Story Generator Service

**Funcionalidade:** Gerar missões, NPCs, histórias

**Exemplo: Gerar Missão**

**Input:**
```json
{
  "campaignContext": "Campanha de D&D 5e em Waterdeep",
  "partyLevel": 5,
  "partySize": 4,
  "preferences": ["mystery", "urban", "intrigue"]
}
```

**Prompt:**
```typescript
const prompt = `
Você é um mestre de RPG experiente criando uma missão para D&D 5e.

Contexto:
- Localização: ${context.location}
- Nível do grupo: ${context.partyLevel}
- Tamanho do grupo: ${context.partySize}
- Preferências: ${context.preferences.join(', ')}

Crie uma missão completa em JSON:
{
  "title": "Título da missão",
  "hook": "Como os jogadores são envolvidos",
  "description": "Descrição completa (300-500 palavras)",
  "objectives": ["objetivo1", "objetivo2"],
  "npcs": [
    {
      "name": "Nome",
      "role": "Papel na missão",
      "personality": "Traços",
      "stats": "Resumo de stats"
    }
  ],
  "encounters": [
    {
      "type": "combat/social/puzzle",
      "description": "Descrição",
      "difficulty": "easy/medium/hard",
      "rewards": "XP e loot"
    }
  ],
  "rewards": {
    "xp": número,
    "gold": número,
    "items": ["item1", "item2"]
  },
  "twists": ["possível twist 1", "possível twist 2"]
}
`;
```

**Custo:** ~$0.20 por missão (mais complexo)

---

#### 4. Dice Roller Service

**Funcionalidade:** Detectar trapaça usando análise estatística

**Não usa OpenAI, usa ML próprio:**

```typescript
// Análise estatística simples
class CheatDetector {
  analyze(userRolls: Roll[]): SuspicionScore {
    const stats = {
      totalRolls: userRolls.length,
      criticals: userRolls.filter(r => r.result === 20).length,
      fails: userRolls.filter(r => r.result === 1).length,
      average: userRolls.reduce((sum, r) => sum + r.result, 0) / userRolls.length
    };
    
    // Probabilidade esperada
    const expectedCritRate = 0.05;  // 5% (1 em 20)
    const actualCritRate = stats.criticals / stats.totalRolls;
    
    // Chi-squared test
    const chiSquared = this.chiSquaredTest(userRolls);
    
    // Score de suspeita (0-1)
    let suspicion = 0;
    
    if (actualCritRate > expectedCritRate * 2) {
      suspicion += 0.3;  // Muitos críticos
    }
    
    if (chiSquared > 30) {
      suspicion += 0.4;  // Distribuição não uniforme
    }
    
    if (stats.average > 11.5) {
      suspicion += 0.3;  // Média muito alta
    }
    
    return {
      score: Math.min(suspicion, 1),
      confidence: stats.totalRolls > 100 ? 'high' : 'medium',
      flags: this.generateFlags(stats)
    };
  }
}
```

**Custo:** $0 (algoritmo próprio)

---

### Controle de Custos

#### 1. Rate Limiting por Tier

```typescript
const AI_LIMITS = {
  free: {
    characterGeneration: 0,  // Sem IA
    battleNarrative: 0,
    storyGeneration: 0
  },
  pro: {
    characterGeneration: 100,  // 100/mês
    battleNarrative: 100,
    storyGeneration: 0  // Não disponível
  },
  master: {
    characterGeneration: 1000,
    battleNarrative: 1000,
    storyGeneration: 100
  }
};

@Injectable()
export class AIRateLimiter {
  async checkLimit(userId: string, feature: string, tier: string): Promise<boolean> {
    const limit = AI_LIMITS[tier][feature];
    const used = await this.redis.get(`ai:${userId}:${feature}:${month}`);
    
    if (used >= limit) {
      throw new RateLimitException(`AI limit reached for ${feature}`);
    }
    
    return true;
  }
  
  async incrementUsage(userId: string, feature: string, cost: number): Promise<void> {
    await this.redis.incr(`ai:${userId}:${feature}:${month}`);
    await this.redis.incrbyfloat(`ai:cost:${userId}:${month}`, cost);
  }
}
```

#### 2. Cache de Respostas

```typescript
@Injectable()
export class AICache {
  async getCached(prompt: string): Promise<string | null> {
    // Hash do prompt
    const key = `ai:cache:${md5(prompt)}`;
    return await this.redis.get(key);
  }
  
  async setCached(prompt: string, response: string, ttl: number = 3600): Promise<void> {
    const key = `ai:cache:${md5(prompt)}`;
    await this.redis.setex(key, ttl, response);
  }
}

// Uso
const cached = await this.aiCache.getCached(prompt);
if (cached) {
  return JSON.parse(cached);  // Economia de $0.10!
}

const response = await openai.chat.completions.create({...});
await this.aiCache.setCached(prompt, JSON.stringify(response));
```

#### 3. Fallback para Geração Simples

```typescript
@Injectable()
export class CharacterGenerator {
  async generate(description: string, tier: string): Promise<Character> {
    if (tier === 'free') {
      // Geração sem IA (algoritmo próprio)
      return this.generateWithoutAI(description);
    }
    
    try {
      // Tenta com IA
      return await this.generateWithAI(description);
    } catch (error) {
      // Fallback se IA falhar
      this.logger.warn('AI failed, using fallback', error);
      return this.generateWithoutAI(description);
    }
  }
  
  private generateWithoutAI(description: string): Character {
    // Algoritmo baseado em regras
    // Mais simples, mas funcional
    return {
      name: this.generateName(),
      stats: this.rollStats(),
      backstory: 'Generated without AI'
    };
  }
}
```

#### 4. Monitoramento de Custos

```typescript
@Injectable()
export class CostMonitor {
  async trackAIUsage(userId: string, feature: string, cost: number): Promise<void> {
    // Salva no Analytics
    await this.analyticsService.track({
      event: 'ai_usage',
      userId,
      feature,
      cost,
      timestamp: new Date()
    });
    
    // Alerta se custo muito alto
    const monthCost = await this.getMonthCost(userId);
    if (monthCost > 10) {  // $10/mês por usuário
      await this.alertService.send({
        type: 'high_ai_cost',
        userId,
        cost: monthCost
      });
    }
  }
}
```

### Qualidade e Segurança

#### 1. Validação de Output

```typescript
@Injectable()
export class AIValidator {
  validateCharacter(character: any): boolean {
    // Valida estrutura
    if (!character.name || !character.stats) {
      throw new ValidationException('Invalid character structure');
    }
    
    // Valida stats (soma deve ser 70-80)
    const statsSum = Object.values(character.stats).reduce((a: number, b: number) => a + b, 0);
    if (statsSum < 70 || statsSum > 80) {
      throw new ValidationException('Invalid stats sum');
    }
    
    // Valida conteúdo inadequado
    if (this.hasInappropriateContent(character.backstory)) {
      throw new ValidationException('Inappropriate content detected');
    }
    
    return true;
  }
  
  private hasInappropriateContent(text: string): boolean {
    const bannedWords = ['...'];  // Lista de palavras proibidas
    return bannedWords.some(word => text.toLowerCase().includes(word));
  }
}
```

#### 2. Moderação de Conteúdo

```typescript
// Usa API de moderação do OpenAI
const moderation = await openai.moderations.create({
  input: character.backstory
});

if (moderation.results[0].flagged) {
  throw new ModerationException('Content flagged by moderation');
}
```

## Alternativas Consideradas

### Alternativa 1: Anthropic Claude

**Vantagens:**
- ✅ Mais barato que GPT-4
- ✅ Contexto maior (100K tokens)
- ✅ Boa qualidade

**Desvantagens:**
- ❌ Menos conhecido
- ❌ Sem geração de imagens
- ❌ API menos madura

**Por que rejeitamos:**
- GPT-4 tem melhor qualidade para nosso caso
- DALL-E é essencial
- Pode adicionar Claude como fallback

### Alternativa 2: Modelos Open Source (Llama, Mistral)

**Vantagens:**
- ✅ Custo zero (self-hosted)
- ✅ Controle total
- ✅ Privacidade

**Desvantagens:**
- ❌ Qualidade inferior
- ❌ Infraestrutura cara (GPUs)
- ❌ Manutenção complexa

**Por que rejeitamos:**
- Qualidade é crítica
- Custo de infra > custo de API
- Pode considerar no futuro

### Alternativa 3: Stable Diffusion (imagens)

**Vantagens:**
- ✅ Mais barato que DALL-E
- ✅ Self-hosted possível

**Desvantagens:**
- ❌ Qualidade inconsistente
- ❌ Precisa fine-tuning
- ❌ Infraestrutura cara

**Por que rejeitamos:**
- DALL-E tem qualidade superior
- $0.04/imagem é aceitável

## Consequências

### Positivas

✅ **Diferenciação Competitiva:**
- Única plataforma com IA integrada
- Valor percebido muito alto

✅ **Experiência do Usuário:**
- Criação de personagem: 30s vs 30min
- Combates mais interessantes
- Conteúdo infinito

✅ **Monetização:**
- Justifica tiers pagos
- Pro: $9.99/mês (custo IA: $3/mês)
- Master: $29.99/mês (custo IA: $10/mês)
- Margem: 50%+

✅ **Escalabilidade:**
- Rate limiting por tier
- Cache reduz custos
- Fallback garante disponibilidade

### Negativas

⚠️ **Custos Variáveis:**
- Difícil prever custos exatos
- Pode explodir se não controlar

**Mitigação:**
- Rate limiting rigoroso
- Monitoramento constante
- Alertas de custo

⚠️ **Dependência de Terceiros:**
- OpenAI pode mudar preços
- API pode ficar indisponível

**Mitigação:**
- Fallback para geração simples
- Cache agressivo
- Considerar alternativas (Claude)

⚠️ **Latência:**
- GPT-4: 5-15s
- DALL-E: 10-20s
- Usuário precisa esperar

**Mitigação:**
- Processamento assíncrono
- Feedback de progresso
- WebSocket para updates

⚠️ **Qualidade Variável:**
- IA pode gerar conteúdo ruim
- Precisa validação

**Mitigação:**
- Validação automática
- Moderação de conteúdo
- Feedback do usuário

## Métricas de Sucesso

| Métrica | Target |
|---------|--------|
| Custo médio por usuário Pro | < $3/mês |
| Custo médio por usuário Master | < $10/mês |
| Taxa de uso de IA (Pro) | > 80% |
| Satisfação com IA | > 4.5/5 |
| Latência média (geração) | < 10s |
| Taxa de cache hit | > 30% |
| Conteúdo flagged | < 1% |

## Referências

- [OpenAI API Documentation](https://platform.openai.com/docs)
- [OpenAI Pricing](https://openai.com/pricing)
- [Best Practices for Prompt Engineering](https://platform.openai.com/docs/guides/prompt-engineering)
- [Content Moderation](https://platform.openai.com/docs/guides/moderation)

## Histórico de Revisões

| Data | Versão | Autor | Descrição |
|------|--------|-------|-----------|
| 2025-10-05 | 1.0 | Laura Fumagalli | Estratégia inicial de integração com IA |

---

**Documentos Relacionados:**
- [ADR 001: Arquitetura de Microserviços](./001-arquitetura-microservicos.md)
- [ADR 002: Estratégia de Dados](./002-estrategia-dados.md)
- [Modelo de Negócio](../BUSINESS_MODEL.md)
