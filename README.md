# 🎲 RPG System - Plataforma Completa para RPG de Mesa

> **Sistema inteligente para criação de personagens, simulação de combates e gerenciamento de campanhas de RPG**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Status](https://img.shields.io/badge/Status-Em%20Desenvolvimento-blue)]()
[![Arquitetura](https://img.shields.io/badge/Arquitetura-Microservi%C3%A7os-green)]()

## 🌟 Visão Geral

O **RPG System** é uma plataforma completa que combina **microserviços**, **inteligência artificial** e **event-driven architecture** para criar uma experiência única para jogadores e mestres de RPG de mesa.

### 🎯 Problema que Resolvemos

- **Jogadores:** Criar personagens é demorado e complexo
- **Mestres:** Preparar sessões requer horas de trabalho
- **Ambos:** Falta de ferramentas integradas e inteligentes

### 💡 Nossa Solução

Uma plataforma que usa **IA** para:
- Gerar personagens a partir de descrições em linguagem natural
- Simular combates com narrativas dinâmicas
- Criar missões e NPCs automaticamente
- Gerenciar campanhas completas
- Detectar trapaças em rolagens de dados

---

## 🏗️ Arquitetura

### Microserviços

```
┌─────────────────────────────────────────────────────────────┐
│                    API Gateway (NestJS)                      │
│  • Autenticação JWT                                         │
│  • Rate Limiting                                            │
│  • Roteamento                                               │
└────┬────────┬────────┬────────┬────────┬────────┬──────────┘
     │        │        │        │        │        │
     ↓        ↓        ↓        ↓        ↓        ↓
┌─────────┐ ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐ ┌──────────┐
│Character│ │Battle│ │Story │ │Dice  │ │Notify│ │Analytics │
│Generator│ │Sim   │ │Gen   │ │Roller│ │      │ │          │
│         │ │      │ │      │ │      │ │      │ │          │
│NestJS   │ │Rust  │ │Python│ │Node  │ │Node  │ │Python    │
│+ OpenAI │ │      │ │+OpenAI│ │      │ │      │ │          │
└────┬────┘ └──┬───┘ └──┬───┘ └──┬───┘ └──┬───┘ └────┬─────┘
     │         │        │        │        │          │
     ↓         ↓        ↓        ↓        ↓          ↓
┌─────────────────────────────────────────────────────────────┐
│                    Event Bus (Kafka)                         │
└─────────────────────────────────────────────────────────────┘
     │         │        │        │        │          │
     ↓         ↓        ↓        ↓        ↓          ↓
┌─────────┐ ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐ ┌──────────┐
│PostgreSQL│ │MongoDB│ │Postgres│Redis │ │SendGrid│TimescaleDB│
└─────────┘ └──────┘ └──────┘ └──────┘ └──────┘ └──────────┘
```

### 📦 Microserviços Detalhados

| Serviço | Responsabilidade | Tech Stack | Database | Por que Microserviço? |
|---------|-----------------|------------|----------|---------------------|
| **API Gateway** | Autenticação, roteamento | NestJS | - | Ponto único de entrada |
| **Character Generator** | Gera personagens com IA | NestJS + OpenAI | PostgreSQL | Processamento pesado (10-30s) |
| **Battle Simulator** | Simula combates | Go | MongoDB | CPU-intensive, concorrência |
| **Story Generator** | Gera histórias/NPCs | Python + OpenAI | PostgreSQL | Usa LLM (caro), rate limiting |
| **Dice Roller** | Rolagem de dados tempo real | Node.js | Redis | Alta frequência, baixa latência |
| **Notification Service** | Emails, push, WebSocket | Node.js | - | Pode falhar isoladamente |
| **Analytics Service** | Métricas e insights | Python | TimescaleDB | Big data, processamento batch |
| **Campaign Manager** | Gerencia campanhas | NestJS | PostgreSQL | Domínio diferente |

---

## 🤖 Inteligência Artificial

### Onde IA Faz Diferença

#### 1. **Geração de Personagens**
```
Input: "Quero um mago elfo velho e sábio, especialista em fogo"

IA processa e gera:
✓ Nome: "Eldrin Flameheart"
✓ Stats otimizados: INT 18, WIS 16, STR 8
✓ Backstory completa e coerente
✓ Spells apropriados: Fireball, Wall of Fire...
✓ Avatar gerado com DALL-E
```

#### 2. **Narrativa de Combates**
```
Input: Grog (Bárbaro) vs Goblin

IA narra:
"Grog avança com sua machado em punho, os músculos tensos.
O goblin tenta esquivar, mas é muito lento.
Com um golpe final devastador, Grog emerge vitorioso!"

+ Animação gerada automaticamente
```

#### 3. **Mestre Virtual**
```
Input: "Preciso de uma missão para nível 5"

IA cria:
✓ Missão completa com objetivos
✓ NPCs com personalidade
✓ Encontros balanceados
✓ Recompensas apropriadas
```

#### 4. **Detecção de Trapaça**
```
IA analisa padrões de rolagem:
- Frequência de críticos
- Distribuição estatística
- Contexto das rolagens

Alerta: "Suspeita de manipulação detectada"
```

---

## 💰 Modelo de Negócio

### 🆓 Free Tier
- 5 personagens
- 10 combates/mês
- Geração básica (sem IA)
- Campanhas públicas

### ⭐ Pro ($9.99/mês)
- **Personagens ilimitados** com IA
- **Combates ilimitados** com narrativa
- **Avatares gerados** por IA
- **Campanhas privadas**
- Suporte prioritário

### 👑 Master ($29.99/mês)
- Tudo do Pro +
- **Mestre Virtual** (IA gera missões)
- **Story Generator** ilimitado
- **Analytics avançado**
- **API access** para integrações
- **Sem anúncios**

### 🏢 Enterprise (Customizado)
- White-label
- Infraestrutura dedicada
- SLA garantido
- Suporte 24/7

---

## 🚀 Funcionalidades

### Para Jogadores

- ✅ **Criação Inteligente de Personagens**
  - Descreva em linguagem natural
  - IA interpreta e gera stats
  - Avatar personalizado com IA
  
- ✅ **Simulador de Combate**
  - Simule lutas antes de jogar
  - Narrativa dinâmica
  - Estatísticas detalhadas

- ✅ **Gerenciador de Fichas**
  - Fichas digitais interativas
  - Histórico de evolução
  - Compartilhamento fácil

- ✅ **Rolador de Dados Inteligente**
  - Física realista
  - Histórico completo
  - Integração com fichas

### Para Mestres

- ✅ **Mestre Virtual (IA)**
  - Gera missões automaticamente
  - Cria NPCs com personalidade
  - Sugere encontros balanceados

- ✅ **Gerenciador de Campanhas**
  - Organize sessões
  - Convide jogadores
  - Log de eventos

- ✅ **Gerador de Histórias**
  - Backstories para NPCs
  - Plots e twists
  - Descrições de locais

- ✅ **Analytics**
  - Estatísticas da campanha
  - Progressão dos jogadores
  - Insights para melhorar sessões

---

## 📚 Documentação

### Arquitetura e Decisões

- [📖 Visão Geral da Arquitetura](./docs/ARCHITECTURE.md)
- [📋 ADRs (Architecture Decision Records)](./docs/adr/README.md)
  - [ADR 001: Arquitetura de Microserviços](./docs/adr/001-arquitetura-microservicos.md)
  - [ADR 002: Estratégia de Dados](./docs/adr/002-estrategia-dados.md)
  - [ADR 003: Integração com IA](./docs/adr/003-integracao-ia.md)
  - [ADR 004: Event-Driven Architecture](./docs/adr/004-event-driven.md)

### Guias Técnicos

- [🔧 Guia de Desenvolvimento](./docs/DEVELOPMENT.md)
- [🎯 Modelo de Negócio](./docs/BUSINESS_MODEL.md)
- [🔐 Segurança e Autenticação](./docs/SECURITY.md)
- [📊 Observabilidade](./docs/OBSERVABILITY.md)

### Microserviços

- [🎭 Character Generator Service](./docs/services/character-generator.md)
- [⚔️ Battle Simulator Service](./docs/services/battle-simulator.md)
- [📖 Story Generator Service](./docs/services/story-generator.md)
- [🎲 Dice Roller Service](./docs/services/dice-roller.md)
- [📢 Notification Service](./docs/services/notification.md)
- [📊 Analytics Service](./docs/services/analytics.md)
- [🗂️ Campaign Manager Service](./docs/services/campaign-manager.md)

---

## 🛠️ Stack Tecnológico

### Backend
- **Node.js/NestJS** - API Gateway, Character Generator, Campaign Manager
- **Go** - Battle Simulator (performance e concorrência)
- **Python** - Story Generator, Analytics (ML/IA)

### Inteligência Artificial
- **OpenAI GPT-4** - Geração de texto e interpretação
- **DALL-E 3** - Geração de avatares
- **Custom ML Models** - Detecção de padrões

### Infraestrutura
- **Kafka** - Event streaming
- **PostgreSQL** - Dados relacionais
- **MongoDB** - Dados de combates (documentos)
- **Redis** - Cache e dados de rolagem
- **TimescaleDB** - Séries temporais (analytics)

### DevOps
- **Docker** - Containerização
- **Kubernetes** - Orquestração
- **GitHub Actions** - CI/CD
- **Prometheus + Grafana** - Observabilidade

---

## 🎯 Roadmap

### ✅ Fase 1: MVP (Q1 2025)
- [x] API Gateway + Auth
- [x] Character Generator (básico)
- [x] Dice Roller
- [x] PostgreSQL setup
- [x] Kafka básico

### 🚧 Fase 2: IA (Q2 2025)
- [ ] Integração OpenAI
- [ ] Geração de personagens com IA
- [ ] DALL-E para avatares
- [ ] Battle Narrator

### 📋 Fase 3: Escalabilidade (Q3 2025)
- [ ] Battle Simulator (Rust)
- [ ] Database per service
- [ ] Redis cache
- [ ] Analytics service

### 🌟 Fase 4: Features Avançadas (Q4 2025)
- [ ] Campaign Manager
- [ ] Story Generator
- [ ] Mestre Virtual
- [ ] Mobile app

---

## 🏃 Quick Start

### Pré-requisitos
- Docker & Docker Compose
- Node.js 20+
- npm ou yarn

### Instalação

```bash
# Clone o repositório
git clone https://github.com/seu-usuario/rpg-system.git
cd rpg-system

# Configure variáveis de ambiente
cp .env.example .env
# Edite .env com suas chaves de API

# Inicie todos os serviços
docker-compose up -d

# Acesse a aplicação
open http://localhost:3000
```

### Desenvolvimento Local

```bash
# API Gateway
cd rpg-system-api
npm install
npm run start:dev

# Character Generator
cd rpg-system-character-generator
npm install
npm run start:dev

# Ver logs do Kafka
docker-compose logs -f kafka
```

---

## 🤝 Contribuindo

Adoramos contribuições! Veja nosso [Guia de Contribuição](./CONTRIBUTING.md).

### Como Contribuir

1. Fork o projeto
2. Crie uma branch (`git checkout -b feature/AmazingFeature`)
3. Commit suas mudanças (`git commit -m 'Add some AmazingFeature'`)
4. Push para a branch (`git push origin feature/AmazingFeature`)
5. Abra um Pull Request

---

## 📄 Licença

Este projeto está sob a **licença MIT**. Veja [LICENSE](./LICENSE) para mais detalhes.

### Por que MIT?

- ✅ **Uso comercial permitido** - Empresas podem usar sem restrições
- ✅ **Modificação livre** - Clientes podem adaptar para suas necessidades
- ✅ **White-label friendly** - Ideal para tier Enterprise
- ✅ **Atrai contribuidores** - Empresas podem contribuir sem medo
- ✅ **Modelo híbrido** - Core open-source, features premium fechadas

**Modelo de negócio:** Vendemos o **serviço (SaaS)**, não o software.

---

## 👥 Time

- **Laura Fumagalli** - Arquiteta de Software - [@laura.fumagalli](https://github.com/laura-fumagalli)

---

## 🙏 Agradecimentos

- OpenAI pela API GPT-4 e DALL-E
- Comunidade RPG por feedback
- Todos os contribuidores

---

## 📞 Contato

- **Website:** [rpgsystem.io](https://rpgsystem.io)
- **Email:** contato@rpgsystem.io
- **Discord:** [discord.gg/rpgsystem](https://discord.gg/rpgsystem)
- **Twitter:** [@rpgsystem](https://twitter.com/rpgsystem)

---

<p align="center">
  Feito com ❤️ e ☕ por desenvolvedores que amam RPG
</p>