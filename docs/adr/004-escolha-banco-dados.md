# ADR 004: Escolha do Banco de Dados - PostgreSQL vs NoSQL

- **Status:** Em Discussão
- **Data:** 2025-10-05
- **Autor:** Laura Fumagalli

## Contexto

O RPG System possui requisitos específicos que precisam ser analisados para escolher o banco de dados adequado:

### Características do Sistema

1. **Modelo de Dados Híbrido:**
   - Dados estruturados: Sistemas, Templates, Personagens
   - Dados semi-estruturados: `regra_geracao` (JSONB)
   - Relacionamentos complexos: Templates → DefinicaoAtributos → InstanciaAtributos

2. **Padrões de Acesso:**
   - **Leitura:** Consultas com JOINs complexos para montar fichas de personagens
   - **Escrita:** Criação de personagens (transacional)
   - **Queries:** Busca por atributos específicos dentro de JSON

3. **Arquitetura:**
   - **API (service-core-api):** CRUD de personagens, consultas complexas
   - **Worker (service-stats-generator):** Leitura de regras, escrita de atributos
   - **Kafka:** Comunicação assíncrona entre serviços

4. **Requisitos Não-Funcionais:**
   - Consistência forte (ACID) para criação de personagens
   - Flexibilidade para adicionar novos sistemas de RPG
   - Performance em queries com múltiplos JOINs
   - Suporte a queries em campos JSON

## Análise Comparativa

### Opção 1: PostgreSQL (Atual)

#### Vantagens

✅ **ACID e Transações:**
- Criação de personagem é transacional (Personagens + InstanciaAtributos)
- Rollback automático em caso de falha
- Consistência garantida entre tabelas relacionadas

✅ **Relacionamentos Complexos:**
- JOINs nativos e eficientes
- Foreign Keys garantem integridade referencial
- Queries complexas para montar fichas:
  ```sql
  SELECT p.*, ia.valor, da.nome_atributo
  FROM Personagens p
  JOIN InstanciaAtributos ia ON p.id = ia.personagem_id
  JOIN DefinicaoAtributos da ON ia.definicao_id = da.id
  WHERE p.id = 'uuid-grog';
  ```

✅ **JSONB Nativo:**
- Melhor dos dois mundos: estrutura + flexibilidade
- Índices GIN para queries em JSON
- Operadores poderosos (`->`, `->>`, `@>`, `?`)
- Exemplo:
  ```sql
  SELECT * FROM DefinicaoAtributos
  WHERE regra_geracao @> '{"metodo": "rolagem_dados"}';
  ```

✅ **Maturidade e Ferramentas:**
- ORM maduro (TypeORM, Prisma)
- Ferramentas de migração robustas
- Backup e replicação consolidados
- Observabilidade (pg_stat_statements)

✅ **Consistência de Dados:**
- Constraints (NOT NULL, UNIQUE, CHECK)
- Triggers para lógica complexa
- Views para abstrair queries

#### Desvantagens

⚠️ **Escalabilidade Horizontal:**
- Mais complexo que NoSQL (sharding manual)
- Replicação read-only é mais simples

⚠️ **Schema Rígido:**
- Migrações necessárias para mudanças estruturais
- Mas JSONB mitiga isso para campos flexíveis

⚠️ **Performance em Escala:**
- JOINs podem ser custosos em volumes muito altos
- Mas otimizável com índices e particionamento

### Opção 2: MongoDB (NoSQL Documento)

#### Vantagens

✅ **Flexibilidade de Schema:**
- Documentos podem ter estruturas diferentes
- Fácil adicionar novos campos

✅ **Escalabilidade Horizontal:**
- Sharding nativo
- Replicação automática

✅ **Documentos Aninhados:**
- Poderia armazenar personagem completo em um documento:
  ```json
  {
    "_id": "uuid-grog",
    "nome": "Grog",
    "template_id": "uuid-dnd",
    "atributos": [
      { "nome": "Força", "valor": 20 },
      { "nome": "Destreza", "valor": 14 }
    ]
  }
  ```

#### Desvantagens

❌ **Sem Transações Multi-Documento (limitadas):**
- Transações existem, mas com overhead
- Não é o forte do MongoDB
- Criação de personagem envolve múltiplas "tabelas"

❌ **Relacionamentos Complexos:**
- JOINs via `$lookup` são ineficientes
- Desnormalização necessária (duplicação de dados)
- Dificulta queries como "todos personagens de um template"

❌ **Integridade Referencial:**
- Sem Foreign Keys nativas
- Validação manual no código
- Risco de dados órfãos

❌ **Modelo Atual Não Se Encaixa:**
- Nosso modelo é **relacional por natureza**
- Separação entre Definição e Instância é fundamental
- Desnormalizar quebraria a filosofia do sistema

### Opção 3: Cassandra (NoSQL Coluna)

#### Vantagens

✅ **Escalabilidade Massiva:**
- Escrita distribuída
- Alta disponibilidade

#### Desvantagens

❌ **Sem JOINs:**
- Tudo precisa ser desnormalizado
- Incompatível com nosso modelo

❌ **Consistência Eventual:**
- Não garante ACID
- Inadequado para criação transacional

❌ **Complexidade:**
- Modelagem de dados complexa
- Overkill para o escopo atual

### Opção 4: Híbrido (PostgreSQL + Redis)

#### Estrutura

- **PostgreSQL:** Dados principais (Sistemas, Templates, Personagens)
- **Redis:** Cache de fichas prontas, sessões

#### Vantagens

✅ **Melhor Performance de Leitura:**
- Cache de personagens completos no Redis
- Reduz queries complexas

✅ **Mantém ACID:**
- PostgreSQL como source of truth
- Redis como camada de performance

#### Desvantagens

⚠️ **Complexidade Operacional:**
- Dois bancos para gerenciar
- Sincronização cache/banco

⚠️ **Invalidação de Cache:**
- Lógica adicional no código
- Risco de dados desatualizados

## Decisão

**Manter PostgreSQL como banco principal, com possível adição de Redis para cache no futuro.**

### Justificativa

1. **Modelo de Dados é Relacional:**
   - Separação entre Definição (Template) e Instância (Personagem)
   - Relacionamentos são fundamentais, não um "extra"
   - Exemplo: `InstanciaAtributos` referencia `DefinicaoAtributos`

2. **ACID é Crítico:**
   - Criação de personagem deve ser atômica
   - Falha ao criar atributo = rollback completo
   - NoSQL não garante isso naturalmente

3. **JSONB Resolve Flexibilidade:**
   - `regra_geracao` em JSONB permite adicionar novos sistemas
   - Não precisamos de schema totalmente flexível
   - Queries em JSON são eficientes com índices GIN

4. **Queries Complexas:**
   - Montar ficha de personagem requer 3-4 JOINs
   - PostgreSQL faz isso nativamente e eficientemente
   - NoSQL exigiria desnormalização massiva

5. **Maturidade do Ecossistema:**
   - TypeORM/Prisma são maduros para PostgreSQL
   - Migrações, seeds, testes são consolidados
   - Observabilidade e debugging são melhores

6. **Escalabilidade Suficiente:**
   - Para MVP e crescimento inicial, PostgreSQL escala bem
   - Read replicas para leitura
   - Particionamento se necessário
   - Se precisar mais, migração para NoSQL é possível (mas improvável)

## Modelagem de Dados

### API (service-core-api)

**Responsabilidades:**
- CRUD de Sistemas, Templates, Personagens
- Consultas de fichas completas (com JOINs)
- Validação de integridade

**Acesso ao Banco:**
- Direto via ORM (TypeORM)
- Transações para operações críticas
- Read replicas para consultas pesadas (futuro)

### Worker (service-stats-generator)

**Responsabilidades:**
- Ler `DefinicaoAtributos` (regras)
- Executar regras (calcular atributos)
- Escrever `InstanciaAtributos`
- Atualizar status do `Personagem`

**Acesso ao Banco:**
- Direto via ORM (TypeORM)
- Transação para garantir atomicidade:
  ```typescript
  await dataSource.transaction(async (manager) => {
    // 1. Criar todas InstanciaAtributos
    await manager.save(InstanciaAtributos, atributos);
    
    // 2. Atualizar status do Personagem
    await manager.update(Personagens, id, { status: 'COMPLETO' });
  });
  ```

### Compartilhamento de Schema

**Problema:** API e Worker precisam do mesmo schema.

**Soluções:**

1. **Shared Library (rpg-system-core):**
   - Entities do TypeORM em `rpg-system-core`
   - API e Worker importam as entities
   - Garante consistência

2. **Migrações Centralizadas:**
   - Migrações em `rpg-system-infra` ou `rpg-system-api`
   - Worker roda as mesmas migrações
   - Versionamento sincronizado

3. **Database-First (Alternativa):**
   - Schema no banco é a fonte da verdade
   - TypeORM gera entities automaticamente
   - Menos recomendado (perde type safety)

**Recomendação:** Opção 1 (Shared Library)

```
rpg-system-core/
├── src/
│   ├── entities/          ← Entities compartilhadas
│   │   ├── sistema.entity.ts
│   │   ├── template.entity.ts
│   │   ├── personagem.entity.ts
│   │   └── ...
│   └── domain/            ← Lógica de domínio
│       └── ...
```

## Estratégia de Migração (Se Necessário no Futuro)

Se o sistema crescer a ponto de PostgreSQL não ser suficiente:

### Cenário 1: Cache com Redis

**Quando:** Leituras de fichas se tornam gargalo

**Implementação:**
- PostgreSQL continua como source of truth
- Redis cacheia fichas completas (JSON)
- Invalidação ao atualizar personagem

**Impacto:** Baixo (adicionar Redis, lógica de cache)

### Cenário 2: Read Replicas

**Quando:** Leituras pesadas afetam escritas

**Implementação:**
- PostgreSQL com replicação streaming
- Escritas no master, leituras nas replicas
- TypeORM suporta múltiplas conexões

**Impacto:** Médio (configuração de infra)

### Cenário 3: Migração para NoSQL (Improvável)

**Quando:** Sharding horizontal é necessário

**Implementação:**
- Migrar apenas `Personagens` e `InstanciaAtributos` para MongoDB
- Manter `Sistemas`, `Templates`, `DefinicaoAtributos` em PostgreSQL
- API faz queries em ambos

**Impacto:** Alto (reescrita de queries, lógica de sync)

## Consequências

### Positivas

✅ **Consistência Garantida:**
- ACID para todas operações críticas
- Integridade referencial nativa

✅ **Queries Eficientes:**
- JOINs otimizados para montar fichas
- Índices em JSONB para queries flexíveis

✅ **Maturidade:**
- Ferramentas consolidadas (TypeORM, migrações)
- Fácil debugging e observabilidade

✅ **Flexibilidade Controlada:**
- JSONB para campos dinâmicos
- Schema estruturado para dados críticos

✅ **Shared Schema:**
- Entities compartilhadas entre API e Worker
- Consistência de tipos (TypeScript)

### Negativas

⚠️ **Escalabilidade Horizontal:**
- Mais complexa que NoSQL
- Mas suficiente para escopo atual

⚠️ **Migrações:**
- Necessárias para mudanças estruturais
- Mas são versionadas e rastreáveis

⚠️ **Shared Library:**
- API e Worker acoplados via `rpg-system-core`
- Mas é acoplamento intencional (DDD)

## Métricas de Sucesso

| Métrica | Target | Como Medir |
|---------|--------|------------|
| Latência de criação de personagem | < 500ms | Logs de transação |
| Latência de consulta de ficha | < 100ms | APM (Application Performance Monitoring) |
| Throughput de escritas | > 100 req/s | Testes de carga |
| Throughput de leituras | > 500 req/s | Testes de carga |
| Tamanho do banco | Monitorar | Métricas do PostgreSQL |

## Referências

- [PostgreSQL JSONB](https://www.postgresql.org/docs/current/datatype-json.html)
- [When to Use PostgreSQL vs MongoDB](https://www.mongodb.com/compare/mongodb-postgresql)
- [TypeORM Transactions](https://typeorm.io/transactions)
- [Shared Database Pattern](https://microservices.io/patterns/data/shared-database.html)
- [Database per Service Pattern](https://microservices.io/patterns/data/database-per-service.html)

## Histórico de Revisões

| Data | Versão | Autor | Descrição |
|------|--------|-------|-----------|
| 2025-10-05 | 1.0 | Laura Fumagalli | Análise inicial PostgreSQL vs NoSQL |

## Aprovações

| Papel | Nome | Data | Status |
|-------|------|------|--------|
| Arquiteto | Laura Fumagalli | 2025-10-05 | 🔄 Em Discussão |

---

**Próximos Passos:**
1. 📋 Validar decisão com métricas de performance
2. 📋 Implementar entities compartilhadas em `rpg-system-core`
3. 📋 Definir estratégia de migrações
4. 📋 Documentar padrões de acesso ao banco
5. 📋 Configurar observabilidade (pg_stat_statements)
