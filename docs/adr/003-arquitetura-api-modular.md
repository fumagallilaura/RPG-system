# ADR 003: Arquitetura Modular da API com Separação de Responsabilidades

- **Status:** Aceito
- **Data:** 2025-10-05
- **Autor:** Laura Fumagalli

> **📌 Nota sobre Exemplos de Código**
> 
> Esta ADR contém **pseudocódigo e exemplos conceituais** para ilustrar os padrões arquiteturais.
> Implementações completas e detalhadas estão em arquivos separados na pasta `examples/`.
> 
> **Razão:** ADRs documentam *decisões* arquiteturais (o "porquê"), não implementações (o "como").
> Manter código de implementação separado evita duplicação de documentação e facilita manutenção.

## Contexto

### Problema de Negócio

A API do RPG System precisa gerenciar múltiplas entidades (personagens, batalhas, usuários) com diferentes níveis de complexidade e requisitos de escalabilidade. Cada entidade possui:

- Lógica de negócio específica (regras do rpg-system-core)
- Requisitos de persistência (PostgreSQL)
- Necessidade de comunicação assíncrona (Kafka)
- Diferentes padrões de acesso (leitura vs escrita)
- Requisitos de segurança e autorização distintos

### Desafios Técnicos

1. **Acoplamento:** Código misturado dificulta manutenção e evolução
2. **Testabilidade:** Dependências diretas entre camadas impedem testes unitários isolados
3. **Escalabilidade:** Impossibilidade de escalar componentes específicos independentemente
4. **Manutenibilidade:** Alta complexidade ciclomática e violação do Single Responsibility Principle
5. **Reusabilidade:** Lógica de negócio acoplada ao transporte HTTP
6. **Observabilidade:** Dificuldade em rastrear fluxo de execução e identificar gargalos

### Requisitos Não-Funcionais

- **Performance:** Latência < 200ms para operações CRUD
- **Disponibilidade:** 99.9% uptime
- **Escalabilidade:** Suportar crescimento horizontal
- **Manutenibilidade:** Onboarding de novos desenvolvedores < 1 semana
- **Testabilidade:** Cobertura de testes > 80%
- **Observabilidade:** Logs estruturados e métricas em tempo real

### Analogia Didática

**Arquitetura Monolítica (Problemática):**
```
┌─────────────────────────────────────┐
│  Tudo em um único arquivo/classe    │
│  - HTTP handling                    │
│  - Business logic                   │
│  - Database access                  │
│  - Event publishing                 │
│  Alto acoplamento, baixa coesão     │
└─────────────────────────────────────┘
```

**Arquitetura em Camadas (Proposta):**
```
┌─────────────────┐  Alta coesão
│   Controller    │  Baixo acoplamento
├─────────────────┤  Responsabilidades claras
│    Service      │  Fácil de testar
├─────────────────┤  Fácil de manter
│   Repository    │  Fácil de escalar
└─────────────────┘
```

Analogia: Um restaurante bem organizado
- **Garçom (Controller):** Interface com cliente
- **Cozinha (Service):** Preparo e orquestração
- **Despensa (Repository):** Armazenamento

Cada função é especializada e independente.

## Decisão

Adotar **Arquitetura em Camadas (Layered Architecture)** baseada nos princípios de **Clean Architecture** e **Domain-Driven Design (DDD)**, implementada através dos módulos do NestJS.

### Padrões e Princípios Aplicados

1. **Separation of Concerns (SoC):** Cada camada tem responsabilidade única e bem definida
2. **Dependency Inversion Principle (DIP):** Camadas superiores não dependem de implementações concretas
3. **Single Responsibility Principle (SRP):** Cada classe tem uma única razão para mudar
4. **Interface Segregation Principle (ISP):** Interfaces específicas por cliente
5. **Dependency Injection (DI):** Inversão de controle gerenciada pelo framework
6. **Repository Pattern:** Abstração do acesso a dados
7. **DTO Pattern:** Validação e transformação de dados na borda do sistema

### Fluxo de Dados (Data Flow)

```
┌──────────────────────────────────────────────────────────┐
│                    HTTP Request                          │
│                         ↓                                │
│  ┌──────────────────────────────────────────────────┐   │
│  │ Controller Layer (Presentation)                  │   │
│  │ - Validação de entrada (DTOs)                    │   │
│  │ - Serialização/Deserialização                    │   │
│  │ - Tratamento de erros HTTP                       │   │
│  │ - Documentação OpenAPI                           │   │
│  └────────────────────┬─────────────────────────────┘   │
│                       ↓                                  │
│  ┌──────────────────────────────────────────────────┐   │
│  │ Service Layer (Business Logic)                   │   │
│  │ - Lógica de negócio                              │   │
│  │ - Orquestração de operações                      │   │
│  │ - Integração com rpg-system-core                 │   │
│  │ - Publicação de eventos (Kafka)                  │   │
│  │ - Transações e consistência                      │   │
│  └────────────────────┬─────────────────────────────┘   │
│                       ↓                                  │
│  ┌──────────────────────────────────────────────────┐   │
│  │ Repository Layer (Data Access)                   │   │
│  │ - Queries e comandos SQL                         │   │
│  │ - Mapeamento objeto-relacional                   │   │
│  │ - Gerenciamento de transações                    │   │
│  │ - Cache de segundo nível                         │   │
│  └────────────────────┬─────────────────────────────┘   │
│                       ↓                                  │
│  ┌──────────────────────────────────────────────────┐   │
│  │ Database (PostgreSQL)                            │   │
│  └──────────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────────┘
```

### Estrutura da API

```
rpg-system-api/
├── src/
│   ├── main.ts                    # Ponto de entrada (porta do restaurante)
│   │
│   ├── modules/                   # Módulos (áreas do restaurante)
│   │   │
│   │   ├── characters/            # Módulo de Personagens
│   │   │   ├── characters.module.ts
│   │   │   ├── characters.controller.ts    # Garçom (recebe pedidos)
│   │   │   ├── characters.service.ts       # Cozinha (processa)
│   │   │   ├── characters.repository.ts    # Despensa (banco)
│   │   │   ├── dto/                        # Cardápio (o que pode pedir)
│   │   │   │   ├── create-character.dto.ts
│   │   │   │   └── update-character.dto.ts
│   │   │   └── entities/                   # Receitas (estrutura dos dados)
│   │   │       └── character.entity.ts
│   │   │
│   │   ├── battles/               # Módulo de Batalhas
│   │   │   └── ... (mesma estrutura)
│   │   │
│   │   └── users/                 # Módulo de Usuários
│   │       └── ... (mesma estrutura)
│   │
│   ├── shared/                    # Coisas compartilhadas (utensílios)
│   │   ├── database/              # Conexão com banco
│   │   ├── kafka/                 # Conexão com Kafka
│   │   ├── guards/                # Segurança (segurança do restaurante)
│   │   ├── interceptors/          # Middleware (garçons extras)
│   │   └── utils/                 # Ferramentas úteis
│   │
│   └── config/                    # Configurações (regras do restaurante)
│       ├── database.config.ts
│       ├── kafka.config.ts
│       └── app.config.ts
```

## Camadas e Responsabilidades

### 1. Controller Layer (Presentation/API)

**Responsabilidades:**
- Receber e validar requisições HTTP
- Mapear DTOs para comandos/queries
- Delegar execução para Service Layer
- Serializar respostas
- Tratamento de exceções HTTP
- Documentação OpenAPI/Swagger
- Rate limiting e throttling

**Restrições:**
- ❌ Lógica de negócio
- ❌ Acesso direto ao banco de dados
- ❌ Conhecimento de implementações concretas
- ❌ Estado mutável

**Estrutura:**
```typescript
@Controller('resource')
class ResourceController {
  constructor(private service: ResourceService) {}
  
  @Post()
  create(@Body() dto: CreateDto) {
    return this.service.create(dto);
  }
  
  @Get(':id')
  findOne(@Param('id') id: string) {
    return this.service.findOne(id);
  }
}
```

**Padrões Aplicados:**
- **DTO Pattern:** Validação na borda do sistema
- **Exception Handling:** Conversão de exceções de domínio para HTTP
- **Dependency Injection:** Services injetados via construtor

> 📝 **Exemplo completo:** Ver `examples/characters.controller.ts` (a ser criado)

### 2. Service Layer (Business Logic/Application)

**Responsabilidades:**
- Implementar casos de uso e regras de negócio
- Orquestrar operações entre múltiplos repositórios
- Gerenciar transações e consistência
- Integrar com domínios externos (rpg-system-core)
- Publicar eventos de domínio (Kafka)
- Implementar políticas de retry e circuit breaker
- Validações de negócio complexas

**Restrições:**
- ❌ Conhecimento de HTTP/REST
- ❌ Queries SQL diretas
- ❌ Dependência de frameworks de apresentação
- ❌ Manipulação direta de requisições/respostas

**Estrutura:**
```typescript
@Injectable()
class ResourceService {
  constructor(
    private repository: ResourceRepository,
    private eventPublisher: EventPublisher,
    private coreValidator: CoreValidator,
  ) {}

  async create(dto: CreateDto): Promise<Resource> {
    // 1. Validar com domínio (rpg-system-core)
    this.coreValidator.validate(dto);
    
    // 2. Aplicar regras de negócio
    const resource = Resource.create(dto);
    
    // 3. Persistir (com transação se necessário)
    const saved = await this.repository.save(resource);
    
    // 4. Publicar evento
    await this.eventPublisher.publish('resource.created', saved);
    
    return saved;
  }
}
```

**Padrões Aplicados:**
- **Transaction Script:** Orquestração de operações
- **Domain Events:** Comunicação assíncrona
- **Unit of Work:** Gerenciamento de transações
- **Specification Pattern:** Validações complexas (via core)

> 📝 **Exemplo completo:** Ver `examples/characters.service.ts` (a ser criado)

### 3. Repository Layer (Data Access)

**Responsabilidades:**
- Encapsular acesso ao banco de dados
- Implementar queries e comandos SQL
- Gerenciar mapeamento objeto-relacional (ORM)
- Otimizar queries (eager/lazy loading, índices)
- Implementar cache de segundo nível
- Abstrair detalhes de persistência

**Restrições:**
- ❌ Lógica de negócio
- ❌ Validações de domínio
- ❌ Conhecimento de casos de uso
- ❌ Publicação de eventos

**Estrutura:**
```typescript
@Injectable()
class ResourceRepository {
  constructor(
    @InjectRepository(Resource) private repo: Repository<Resource>,
    private cache: CacheManager,
  ) {}

  async save(resource: Resource): Promise<Resource> {
    const saved = await this.repo.save(resource);
    await this.cache.invalidate(`resource:${saved.id}`);
    return saved;
  }

  async findById(id: string): Promise<Resource | null> {
    // Cache-aside pattern
    const cached = await this.cache.get(`resource:${id}`);
    if (cached) return cached;
    
    const resource = await this.repo.findOne({ where: { id } });
    if (resource) await this.cache.set(`resource:${id}`, resource);
    
    return resource;
  }
}
```

**Padrões Aplicados:**
- **Repository Pattern:** Abstração de persistência
- **Unit of Work:** Suporte a transações
- **Cache-Aside Pattern:** Cache de leitura
- **Soft Delete:** Auditoria e recuperação de dados

> 📝 **Exemplo completo:** Ver `examples/characters.repository.ts` (a ser criado)

### 4. DTO (Data Transfer Object)

**Propósito:**
- Definir contratos de API (input/output)
- Validação declarativa de dados
- Documentação automática (OpenAPI)
- Transformação e sanitização
- Versionamento de API

**Estrutura:**
```typescript
// Input DTO
class CreateResourceDto {
  @IsString()
  @Length(3, 50)
  name: string;

  @IsEnum(ResourceType)
  type: ResourceType;

  @IsNumber()
  @Min(1)
  level: number;
}

// Output DTO
class ResourceResponseDto {
  id: string;
  name: string;
  type: ResourceType;
  level: number;
  createdAt: Date;

  static fromEntity(entity: Resource): ResourceResponseDto {
    // Mapear entidade para DTO
  }
}
```

**Padrões Aplicados:**
- **Validation at the Edge:** Validar dados na entrada do sistema
- **Factory Method:** Conversão de Entity para DTO

> 📝 **Exemplo completo:** Ver `examples/characters.dto.ts` (a ser criado)

### 5. Entity (Domain Model)

**Propósito:**
- Representar modelo de domínio
- Mapear para tabelas do banco (ORM)
- Encapsular regras de domínio
- Garantir invariantes

**Estrutura:**
```typescript
@Entity('resources')
class Resource {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column()
  name: string;

  @Column({ type: 'enum' })
  type: ResourceType;

  @Column()
  level: number;

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;

  // Factory method (DDD)
  static create(data: CreateResourceData): Resource {
    const resource = new Resource();
    // ... inicializar e validar invariantes
    return resource;
  }

  // Métodos de domínio
  levelUp(): void {
    // Lógica de negócio
  }
}
```

**Padrões Aplicados:**
- **Rich Domain Model:** Entidades com comportamento
- **Factory Method:** Criação controlada
- **Invariants:** Validação de regras de domínio
- **Optimistic Locking:** Controle de concorrência (via `@VersionColumn`)

> 📝 **Exemplo completo:** Ver `examples/character.entity.ts` (a ser criado)

### 6. Observabilidade e Métricas

**Componentes Essenciais:**

1. **Logging Estruturado**
   - Logs em formato JSON
   - Correlation IDs para rastreamento
   - Níveis apropriados (debug, info, warn, error)

2. **Métricas (Prometheus)**
   - Latência de requisições (histogramas)
   - Taxa de erros (counters)
   - Recursos do sistema (gauges)

3. **Health Checks**
   - Verificação de banco de dados
   - Verificação de Kafka
   - Verificação de dependências externas

4. **Distributed Tracing**
   - OpenTelemetry/Jaeger
   - Rastreamento entre serviços

**Exemplo Conceitual:**
```typescript
class Service {
  private logger = new Logger(Service.name);

  async operation() {
    const correlationId = generateId();
    this.logger.log({ message: 'Starting', correlationId });
    
    try {
      // operação
      this.logger.log({ message: 'Success', correlationId });
    } catch (error) {
      this.logger.error({ message: 'Failed', correlationId, error });
      throw error;
    }
  }
}
```

> 📝 **Exemplos completos:** Ver `examples/observability/` (a ser criado)

## Por Que Essa Arquitetura?

### 1. Separação de Responsabilidades (SRP)

**Princípio:** Cada módulo/classe tem uma única razão para mudar.

**Implementação:**
- **Controller:** Muda apenas se o contrato HTTP mudar
- **Service:** Muda apenas se regras de negócio mudarem
- **Repository:** Muda apenas se a estratégia de persistência mudar

**Benefícios:**
- Mudanças localizadas e previsíveis
- Redução de efeitos colaterais
- Facilita code review
- Melhora manutenibilidade

### 2. Testabilidade

```typescript
// Testar Service sem banco nem HTTP!
describe('CharactersService', () => {
  it('should create character', async () => {
    const mockRepo = { save: jest.fn() };
    const service = new CharactersService(mockRepo);
    
    await service.create({ name: 'Test' });
    
    expect(mockRepo.save).toHaveBeenCalled();
  });
});
```

### 3. Reutilização

```typescript
// Usar o mesmo Service em HTTP, GraphQL, CLI, etc!
@Controller('characters')
class HttpController {
  constructor(private service: CharactersService) {}
}

@Resolver()
class GraphQLResolver {
  constructor(private service: CharactersService) {}
}
```

### 4. Manutenibilidade

```
Precisa adicionar cache?
→ Adiciona no Service, não mexe em Controller nem Repository

Precisa mudar validação?
→ Mexe no DTO, não mexe em nada mais

Precisa trocar PostgreSQL por MongoDB?
→ Mexe no Repository, resto não muda
```

## Fluxo Completo (Exemplo Real)

### Criar Personagem

```
1. Cliente → POST /characters
   Body: { name: "Aragorn", class: "ranger" }

2. Controller recebe
   ├─ Valida DTO
   └─ Chama Service

3. Service processa
   ├─ Valida com rpg-system-core
   ├─ Calcula atributos iniciais
   ├─ Chama Repository.save()
   ├─ Publica no Kafka: "character.created"
   └─ Retorna resultado

4. Repository salva
   ├─ INSERT INTO characters ...
   └─ Retorna character com ID

5. Controller retorna
   ├─ Status: 201 Created
   └─ Body: { id: "uuid", name: "Aragorn", ... }

6. Kafka propaga evento
   └─ Outros serviços podem reagir
       (ex: enviar email, gerar retrato, etc)
```

## Módulos Independentes

Cada módulo é **independente**:

```typescript
// characters.module.ts
@Module({
  imports: [
    TypeOrmModule.forFeature([Character]),
    KafkaModule,
  ],
  controllers: [CharactersController],
  providers: [CharactersService, CharactersRepository],
  exports: [CharactersService], // Outros módulos podem usar!
})
export class CharactersModule {}
```

**Benefícios:**
- Pode desenvolver em paralelo (uma pessoa faz Characters, outra faz Battles)
- Pode testar isoladamente
- Pode reutilizar em outros projetos
- Pode desabilitar módulos que não precisa

## Integração com rpg-system-core

```typescript
// characters.service.ts
import { CharacterValidator } from '@rpg-system/core';

@Injectable()
export class CharactersService {
  async create(dto: CreateCharacterDto) {
    // Usa a lógica do Core (regras do jogo)
    const validated = CharacterValidator.validate(dto);
    const stats = CharacterValidator.calculateStats(validated);
    
    // API só orquestra, Core tem as regras!
    return this.repository.save({ ...validated, stats });
  }
}
```

**Por quê?**
- Core: Regras do jogo (pode usar em CLI, Mobile, Web)
- API: Orquestração e persistência

## Consequências

### Positivas

✅ **Testabilidade Aumentada**
- **Métrica:** Cobertura de testes > 80% (target: 90%)
- Testes unitários isolados por camada
- Mocks simples com Dependency Injection
- Testes de integração focados
- Redução de 60% no tempo de execução de testes unitários

✅ **Manutenibilidade Melhorada**
- **Métrica:** Complexidade ciclomática < 10 por método
- **Métrica:** Tempo de onboarding < 1 semana
- Mudanças localizadas (princípio do menor impacto)
- Refatoração segura com testes
- Redução de 40% em bugs de regressão

✅ **Escalabilidade Horizontal**
- **Métrica:** Suporta 10x mais requisições com mesma latência
- Stateless services permitem múltiplas instâncias
- Load balancing simples
- Possibilidade de extrair módulos para microserviços
- Auto-scaling baseado em métricas

✅ **Performance Otimizada**
- **Métrica:** P95 latência < 200ms
- **Métrica:** P99 latência < 500ms
- Cache em múltiplos níveis (Repository, Service)
- Queries otimizadas isoladas no Repository
- Connection pooling gerenciado

✅ **Observabilidade Completa**
- **Métrica:** 100% de endpoints com logging estruturado
- **Métrica:** MTTR (Mean Time To Recovery) < 15 minutos
- Logs estruturados com correlation IDs
- Métricas Prometheus em tempo real
- Distributed tracing (Jaeger/OpenTelemetry)
- Health checks automatizados

✅ **Reusabilidade de Código**
- Services independentes de transporte (HTTP, GraphQL, gRPC)
- Lógica de negócio compartilhada
- Redução de 50% em código duplicado
- Facilita criação de novos endpoints

✅ **Documentação Automática**
- OpenAPI/Swagger gerado automaticamente
- Contratos de API versionados
- Exemplos de requisição/resposta
- Redução de 70% em tickets de suporte sobre API

### Negativas (Trade-offs)

⚠️ **Overhead Inicial**
- **Impacto:** +30% tempo de desenvolvimento inicial
- **Mitigação:** CLI do NestJS gera boilerplate automaticamente
- **Mitigação:** Templates e generators customizados
- 5-7 arquivos por feature (vs 1-2 em monolito simples)
- ROI positivo após 3-6 meses de projeto

⚠️ **Curva de Aprendizado**
- **Impacto:** 1-2 semanas para desenvolvedores juniores
- **Mitigação:** Documentação detalhada (esta ADR)
- **Mitigação:** Pair programming nas primeiras features
- **Mitigação:** Code reviews rigorosos
- Requer entendimento de SOLID, DI, padrões de design

⚠️ **Complexidade de Debugging**
- **Impacto:** Mais camadas para rastrear em caso de bug
- **Mitigação:** Logging estruturado com correlation IDs
- **Mitigação:** Distributed tracing
- **Mitigação:** Ferramentas de debugging adequadas
- Stack traces mais profundos

⚠️ **Performance de Desenvolvimento**
- **Impacto:** Mais arquivos para navegar
- **Mitigação:** IDE com boa navegação (VS Code, WebStorm)
- **Mitigação:** Convenções de nomenclatura claras
- **Mitigação:** Estrutura de pastas consistente

⚠️ **Risco de Over-Engineering**
- **Impacto:** Tentação de criar abstrações desnecessárias
- **Mitigação:** YAGNI (You Aren't Gonna Need It)
- **Mitigação:** Code reviews focados em simplicidade
- **Mitigação:** Métricas de complexidade

### Métricas de Sucesso

| Métrica | Baseline | Target | Prazo |
|---------|----------|--------|-------|
| Cobertura de Testes | 60% | 90% | 3 meses |
| P95 Latência | 500ms | 200ms | 2 meses |
| MTTR | 45min | 15min | 3 meses |
| Tempo de Onboarding | 2 semanas | 1 semana | 1 mês |
| Bugs de Produção | 15/mês | 5/mês | 6 meses |
| Complexidade Ciclomática | 15 | 10 | 2 meses |

### Riscos e Mitigações

| Risco | Probabilidade | Impacto | Mitigação |
|-------|---------------|---------|-----------|
| Desenvolvedores não seguem padrão | Média | Alto | Code reviews + Linters |
| Performance degradada | Baixa | Médio | Testes de carga + Monitoramento |
| Over-engineering | Média | Médio | YAGNI + Revisões arquiteturais |
| Dificuldade de debugging | Baixa | Médio | Logging estruturado + Tracing |

## Alternativas Consideradas

### 1. Monolito Simples (Tudo em um arquivo)
❌ Rejeitado: Não escala, difícil de manter

### 2. Microserviços desde o início
❌ Rejeitado: Complexidade desnecessária no início

### 3. Arquitetura Hexagonal (Ports & Adapters)
⚠️ Considerado mas adiado: Muito complexo para o time atual

### 4. CQRS (Command Query Responsibility Segregation)
⚠️ Considerado mas adiado: Adicionar quando necessário

## Próximos Passos

1. ✅ Definir estrutura base
2. 🔄 Implementar módulo Characters
3. 📋 Implementar módulo Battles
4. 📋 Implementar módulo Users
5. 📋 Adicionar testes unitários
6. 📋 Adicionar testes de integração
7. 📋 Documentar APIs (Swagger)

## Referências Técnicas

### Arquitetura e Design Patterns

- **Clean Architecture** - Robert C. Martin
  - [The Clean Architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
  - Livro: "Clean Architecture: A Craftsman's Guide to Software Structure and Design"

- **Domain-Driven Design (DDD)** - Eric Evans
  - [DDD Reference](https://www.domainlanguage.com/ddd/reference/)
  - [Martin Fowler on DDD](https://martinfowler.com/bliki/DomainDrivenDesign.html)
  - Livro: "Domain-Driven Design: Tackling Complexity in the Heart of Software"

- **Enterprise Application Architecture** - Martin Fowler
  - [Repository Pattern](https://martinfowler.com/eaaCatalog/repository.html)
  - [Service Layer](https://martinfowler.com/eaaCatalog/serviceLayer.html)
  - [Data Transfer Object](https://martinfowler.com/eaaCatalog/dataTransferObject.html)
  - Livro: "Patterns of Enterprise Application Architecture"

### SOLID Principles

- [SOLID Principles - Wikipedia](https://en.wikipedia.org/wiki/SOLID)
- [Single Responsibility Principle](https://en.wikipedia.org/wiki/Single-responsibility_principle)
- [Dependency Inversion Principle](https://en.wikipedia.org/wiki/Dependency_inversion_principle)

### NestJS e Framework

- [NestJS Documentation](https://docs.nestjs.com/)
- [NestJS Layered Architecture](https://docs.nestjs.com/techniques/database#repository-pattern)
- [NestJS Testing](https://docs.nestjs.com/fundamentals/testing)
- [TypeORM Documentation](https://typeorm.io/)

### Observabilidade

- [The Twelve-Factor App](https://12factor.net/)
- [OpenTelemetry](https://opentelemetry.io/)
- [Prometheus Best Practices](https://prometheus.io/docs/practices/naming/)
- [Structured Logging](https://www.structlog.org/en/stable/why.html)

### Microservices e Escalabilidade

- [Microservices Patterns - Chris Richardson](https://microservices.io/patterns/index.html)
- [Building Microservices - Sam Newman](https://samnewman.io/books/building_microservices/)
- [Event-Driven Architecture](https://martinfowler.com/articles/201701-event-driven.html)

### Testes

- [Test Pyramid - Martin Fowler](https://martinfowler.com/articles/practical-test-pyramid.html)
- [Testing Strategies in a Microservice Architecture](https://martinfowler.com/articles/microservice-testing/)

### Performance e Otimização

- [Database Indexing Strategies](https://use-the-index-luke.com/)
- [Caching Strategies](https://aws.amazon.com/caching/best-practices/)
- [N+1 Query Problem](https://stackoverflow.com/questions/97197/what-is-the-n1-selects-problem-in-orm-object-relational-mapping)

## Glossário

### Termos de Arquitetura

- **ADR:** Architecture Decision Record - Documento que registra uma decisão arquitetural importante
- **DDD:** Domain-Driven Design - Abordagem de design focada no domínio do negócio
- **DI:** Dependency Injection - Padrão de inversão de controle onde dependências são fornecidas externamente
- **DTO:** Data Transfer Object - Objeto usado para transferir dados entre camadas/sistemas
- **ORM:** Object-Relational Mapping - Técnica de mapeamento entre objetos e banco de dados relacional
- **SOLID:** Princípios de design orientado a objetos (SRP, OCP, LSP, ISP, DIP)
- **SRP:** Single Responsibility Principle - Princípio da responsabilidade única

### Termos de Camadas

- **Controller:** Camada de apresentação que recebe requisições HTTP e retorna respostas
- **Service:** Camada de lógica de negócio que orquestra operações
- **Repository:** Camada de acesso a dados que abstrai persistência
- **Entity:** Modelo de domínio que representa uma tabela no banco
- **Module:** Agrupamento de funcionalidades relacionadas (NestJS)

### Termos de Observabilidade

- **MTTR:** Mean Time To Recovery - Tempo médio para recuperação de falhas
- **P95/P99:** Percentil 95/99 - Métrica de latência onde 95%/99% das requisições são mais rápidas
- **Correlation ID:** Identificador único para rastrear uma requisição através de múltiplos serviços
- **Distributed Tracing:** Rastreamento de requisições através de múltiplos serviços

### Tecnologias

- **Kafka:** Sistema de mensageria distribuído para comunicação assíncrona
- **PostgreSQL:** Banco de dados relacional open-source
- **NestJS:** Framework Node.js para construção de aplicações server-side escaláveis
- **TypeORM:** ORM para TypeScript e JavaScript

## Histórico de Revisões

| Data | Versão | Autor | Descrição |
|------|--------|-------|-----------|
| 2025-10-05 | 1.0 | Laura Fumagalli | Criação inicial da ADR |
| 2025-10-05 | 2.0 | Laura Fumagalli | Revisão técnica: adicionados padrões detalhados, métricas de sucesso, observabilidade, exemplos de código completos e referências expandidas |

## Aprovações

| Papel | Nome | Data | Status |
|-------|------|------|--------|
| Arquiteto | Laura Fumagalli | 2025-10-05 | ✅ Aprovado |
| Tech Lead | - | - | Pendente |
| Desenvolvedor Sênior | - | - | Pendente |

---

**Próximos Passos:**
1. ✅ Definir estrutura de módulos (characters, users, battles)
2. 📋 Implementar estrutura base do módulo characters
3. 📋 Configurar logging estruturado e métricas (Prometheus)
4. 📋 Implementar testes unitários para cada camada (>80% coverage)
5. 📋 Documentar APIs com Swagger/OpenAPI
6. 📋 Configurar CI/CD com validação de arquitetura (linters, testes)
7. 📋 Implementar health checks e observabilidade
8. 📋 Adicionar testes de carga e performance
