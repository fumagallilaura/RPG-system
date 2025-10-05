# 💰 Modelo de Negócio - RPG System

> **Estratégia de monetização e justificativa para arquitetura de microserviços**

## 📊 Visão Geral

O RPG System adota um modelo **Freemium** com tiers de assinatura, onde funcionalidades avançadas (especialmente IA) são monetizadas.

---

## 🎯 Tiers de Assinatura

### 🆓 Free Tier

**Preço:** Gratuito

**Limites:**
- 5 personagens
- 10 combates/mês
- Sem IA (geração manual)
- Campanhas públicas apenas
- Anúncios

**Funcionalidades:**
- ✅ Criação básica de personagens
- ✅ Rolador de dados
- ✅ Fichas digitais
- ✅ Simulador de combate (sem narrativa)
- ✅ Participar de campanhas públicas

**Objetivo:**
- Aquisição de usuários
- Demonstrar valor do produto
- Converter para Pro

**Custos Estimados:**
- $0.10/usuário/mês (infraestrutura)
- Sem custos de IA

---

### ⭐ Pro Tier

**Preço:** $9.99/mês ou $99/ano (2 meses grátis)

**Limites:**
- Personagens ilimitados
- Combates ilimitados
- 100 gerações com IA/mês
- Campanhas privadas ilimitadas

**Funcionalidades:**
- ✅ Tudo do Free +
- ✅ **Geração de personagens com IA**
  - Interpreta descrição em linguagem natural
  - Gera stats otimizados
  - Cria backstory completa
- ✅ **Avatares gerados por IA (DALL-E)**
- ✅ **Narrativa de combates com IA**
- ✅ **Campanhas privadas**
- ✅ **Sem anúncios**
- ✅ **Suporte prioritário**
- ✅ **Histórico completo**

**Objetivo:**
- Principal fonte de receita
- Usuários engajados
- Uso moderado de IA

**Custos Estimados:**
- $2.00/usuário/mês (infraestrutura)
- $3.00/usuário/mês (IA - OpenAI)
- **Margem:** $4.99/usuário/mês (50%)

---

### 👑 Master Tier

**Preço:** $29.99/mês ou $299/ano (2 meses grátis)

**Limites:**
- Tudo ilimitado
- 1000 gerações com IA/mês
- API access (10k requests/mês)

**Funcionalidades:**
- ✅ Tudo do Pro +
- ✅ **Mestre Virtual (IA)**
  - Gera missões automaticamente
  - Cria NPCs com personalidade
  - Sugere encontros balanceados
- ✅ **Story Generator ilimitado**
  - Backstories para NPCs
  - Plots e twists
  - Descrições de locais
- ✅ **Analytics avançado**
  - Estatísticas da campanha
  - Progressão dos jogadores
  - Insights para melhorar sessões
- ✅ **API Access**
  - Integração com Roll20, Foundry VTT
  - Webhooks customizados
- ✅ **Prioridade máxima**
- ✅ **Beta features**

**Objetivo:**
- Mestres profissionais
- Power users
- Integrações

**Custos Estimados:**
- $5.00/usuário/mês (infraestrutura)
- $10.00/usuário/mês (IA - uso intenso)
- **Margem:** $14.99/usuário/mês (50%)

---

### 🏢 Enterprise Tier

**Preço:** Customizado (a partir de $499/mês)

**Funcionalidades:**
- ✅ Tudo do Master +
- ✅ **White-label**
  - Sua marca, sua URL
  - Customização completa
- ✅ **Infraestrutura dedicada**
  - Servidores isolados
  - Performance garantida
- ✅ **SLA 99.9%**
- ✅ **Suporte 24/7**
- ✅ **Onboarding dedicado**
- ✅ **Custom features**

**Objetivo:**
- Editoras de RPG
- Plataformas grandes
- Eventos e convenções

**Custos Estimados:**
- Variável (infraestrutura dedicada)
- **Margem:** 60-70%

---

## 📈 Projeção de Receita

### Cenário Conservador (Ano 1)

| Tier | Usuários | Preço | Receita/Mês | Receita/Ano |
|------|----------|-------|-------------|-------------|
| Free | 10,000 | $0 | $0 | $0 |
| Pro | 500 | $9.99 | $4,995 | $59,940 |
| Master | 50 | $29.99 | $1,500 | $17,994 |
| Enterprise | 2 | $500 | $1,000 | $12,000 |
| **Total** | **10,552** | - | **$7,495** | **$89,934** |

**Custos Estimados:**
- Infraestrutura: $2,000/mês ($24,000/ano)
- IA (OpenAI): $1,500/mês ($18,000/ano)
- Equipe (2 devs): $10,000/mês ($120,000/ano)
- Marketing: $1,000/mês ($12,000/ano)
- **Total:** $14,500/mês ($174,000/ano)

**Resultado Ano 1:** -$84,066 (investimento)

### Cenário Otimista (Ano 2)

| Tier | Usuários | Preço | Receita/Mês | Receita/Ano |
|------|----------|-------|-------------|-------------|
| Free | 50,000 | $0 | $0 | $0 |
| Pro | 5,000 | $9.99 | $49,950 | $599,400 |
| Master | 500 | $29.99 | $14,995 | $179,940 |
| Enterprise | 10 | $750 | $7,500 | $90,000 |
| **Total** | **55,510** | - | **$72,445** | **$869,340** |

**Custos Estimados:**
- Infraestrutura: $10,000/mês ($120,000/ano)
- IA (OpenAI): $15,000/mês ($180,000/ano)
- Equipe (5 devs): $25,000/mês ($300,000/ano)
- Marketing: $5,000/mês ($60,000/ano)
- **Total:** $55,000/mês ($660,000/ano)

**Resultado Ano 2:** +$209,340 (lucro)

---

## 🎯 Justificativa de Microserviços

### Por que Microserviços Fazem Sentido Financeiramente

#### 1. **Custo Variável por Tier**

```
Free Tier:
├─ API Gateway ✅ (necessário)
├─ Character Generator ✅ (sem IA)
├─ Dice Roller ✅ (necessário)
├─ Campaign Manager ✅ (público apenas)
└─ Outros serviços ❌ (desligados)

Pro Tier:
├─ API Gateway ✅
├─ Character Generator ✅ (com IA)
├─ Battle Simulator ✅ (com narrativa)
├─ Dice Roller ✅
├─ Campaign Manager ✅
└─ Notification Service ✅

Master Tier:
├─ Todos os serviços ✅
└─ Analytics Service ✅ (exclusivo)
```

**Economia:**
- Free users não usam serviços caros (IA)
- Pro users usam moderadamente
- Master users pagam pelo uso intenso

#### 2. **Escalabilidade Independente**

```
Serviço mais usado: Dice Roller (milhares/segundo)
├─ Escala: 10 instâncias
├─ Custo: $100/mês

Serviço menos usado: Story Generator (dezenas/hora)
├─ Escala: 1 instância
├─ Custo: $20/mês

Economia: $180/mês vs $200/mês (monolito)
```

#### 3. **Rate Limiting por Serviço**

```typescript
// Character Generator
const limits = {
  free: 0,  // Sem IA
  pro: 100,  // 100 gerações/mês
  master: 1000  // 1000 gerações/mês
};

// Controla custos de OpenAI
// Pro: $3/usuário/mês
// Master: $10/usuário/mês
```

#### 4. **Fail Gracefully**

```
Se Story Generator cai:
├─ Free users: Não percebem (não usam)
├─ Pro users: Não percebem (não usam)
├─ Master users: Recebem erro, mas resto funciona
└─ Economia: Não precisa redundância cara
```

---

## 🤖 Justificativa de IA

### Por que IA Justifica o Preço

#### 1. **Valor Percebido**

**Sem IA (Free):**
- Criar personagem: 30-60 minutos
- Preparar sessão: 2-4 horas
- Criar NPCs: 15-30 minutos cada

**Com IA (Pro/Master):**
- Criar personagem: 30 segundos
- Preparar sessão: 5-10 minutos
- Criar NPCs: 10 segundos cada

**Economia de tempo:** 10-20 horas/mês
**Valor:** $100-200/mês (se fosse pagar alguém)
**Preço:** $9.99-29.99/mês

**ROI para usuário:** 5-10x

#### 2. **Diferenciação**

Concorrentes:
- D&D Beyond: Sem IA ($6/mês)
- Roll20: Sem IA (grátis)
- Foundry VTT: Sem IA ($50 one-time)

**RPG System:** Única plataforma com IA integrada

#### 3. **Custos Justificados**

```
OpenAI GPT-4:
├─ Geração de personagem: $0.10
├─ Narrativa de combate: $0.05
├─ Geração de missão: $0.20
└─ Total: $0.35/geração

DALL-E 3:
└─ Avatar: $0.04/imagem

Pro user (100 gerações/mês):
├─ GPT-4: $35
├─ DALL-E: $4
└─ Total: $39/mês

Preço: $9.99/mês
Subsídio: $29/mês (financiado por Master users)
```

---

## 📊 Métricas de Sucesso

### KPIs Principais

| Métrica | Ano 1 | Ano 2 | Ano 3 |
|---------|-------|-------|-------|
| **MAU (Monthly Active Users)** | 10k | 50k | 200k |
| **Conversão Free → Pro** | 5% | 10% | 15% |
| **Conversão Pro → Master** | 10% | 15% | 20% |
| **Churn Rate** | 15% | 10% | 5% |
| **LTV (Lifetime Value)** | $60 | $120 | $240 |
| **CAC (Customer Acquisition Cost)** | $20 | $15 | $10 |
| **LTV/CAC Ratio** | 3:1 | 8:1 | 24:1 |

### Métricas de Produto

| Métrica | Target |
|---------|--------|
| Personagens criados com IA/mês | 10,000 |
| Combates simulados/mês | 50,000 |
| Rolagens de dados/mês | 1,000,000 |
| Campanhas ativas | 5,000 |
| Tempo médio de sessão | 45min |
| NPS (Net Promoter Score) | > 50 |

---

## 🚀 Estratégia de Go-to-Market

### Fase 1: Beta Fechado (3 meses)

**Objetivo:** Validar produto e IA

- 100 usuários selecionados
- Grátis (feedback em troca)
- Iterar rapidamente

### Fase 2: Beta Aberto (3 meses)

**Objetivo:** Crescer base de usuários

- Free tier disponível
- Pro tier com 50% desconto
- Marketing em comunidades RPG

### Fase 3: Lançamento Oficial (6 meses)

**Objetivo:** Monetização

- Todos os tiers disponíveis
- Preços finais
- Marketing agressivo

### Fase 4: Escala (12+ meses)

**Objetivo:** Crescimento exponencial

- Parcerias com editoras
- Integrações com plataformas
- Expansão internacional

---

## 💡 Oportunidades Futuras

### 1. Marketplace

- Usuários vendem personagens, missões, NPCs
- RPG System fica com 30%
- Receita adicional: $10k-50k/mês

### 2. API para Terceiros

- Desenvolvedores integram com suas ferramentas
- $99/mês por app
- Receita adicional: $5k-20k/mês

### 3. Eventos e Convenções

- Licença temporária para eventos
- $500-2000/evento
- Receita adicional: $10k-30k/mês

### 4. Conteúdo Premium

- Sistemas oficiais (D&D, Pathfinder)
- $4.99-9.99 one-time
- Receita adicional: $20k-100k/mês

---

## 🎯 Conclusão

O modelo de negócio **justifica completamente** a arquitetura de microserviços e integração com IA:

✅ **Microserviços:** Permitem escalar e cobrar por funcionalidade
✅ **IA:** Diferenciação clara e valor percebido alto
✅ **Freemium:** Aquisição de usuários e conversão gradual
✅ **Tiers:** Monetização de diferentes perfis de usuário

**ROI esperado:** Positivo em 18-24 meses
**Potencial de mercado:** $10M+ ARR em 5 anos

---

**Documentos Relacionados:**
- [Arquitetura](./ARCHITECTURE.md)
- [ADR 001: Arquitetura de Microserviços](./adr/001-arquitetura-microservicos.md)
- [ADR 003: Integração com IA](./adr/003-integracao-ia.md)
