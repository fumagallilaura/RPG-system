# Arquitetura da API - Guia Visual

## 🎯 Visão Geral Simples

```
┌─────────────────────────────────────────────────────────────┐
│                    Cliente (Frontend/Postman)               │
└────────────────────────┬────────────────────────────────────┘
                         │ HTTP Request
                         ▼
┌─────────────────────────────────────────────────────────────┐
│                      CAMADA 1: Controller                    │
│  "O Garçom - Recebe pedidos e entrega respostas"           │
│                                                              │
│  ✅ Recebe HTTP                                             │
│  ✅ Valida formato                                          │
│  ✅ Passa para Service                                      │
│  ❌ NÃO tem lógica de negócio                              │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│                      CAMADA 2: Service                       │
│  "A Cozinha - Processa e orquestra"                        │
│                                                              │
│  ✅ Lógica de negócio                                       │
│  ✅ Orquestra operações                                     │
│  ✅ Usa rpg-system-core                                     │
│  ✅ Publica eventos Kafka                                   │
│  ❌ NÃO acessa banco direto                                │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│                    CAMADA 3: Repository                      │
│  "A Despensa - Gerencia dados"                             │
│                                                              │
│  ✅ Queries SQL                                             │
│  ✅ Transações                                              │
│  ✅ CRUD no banco                                           │
│  ❌ NÃO tem lógica de negócio                              │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│                      PostgreSQL Database                     │
└─────────────────────────────────────────────────────────────┘
```

## 📦 Estrutura de um Módulo

```
characters/
│
├── characters.module.ts          # Configuração do módulo
│   └─ Define: Controllers, Services, Repositories
│
├── characters.controller.ts      # CAMADA 1: HTTP
│   ├─ @Get()
│   ├─ @Post()
│   ├─ @Put()
│   └─ @Delete()
│
├── characters.service.ts         # CAMADA 2: Lógica
│   ├─ create()
│   ├─ findAll()
│   ├─ findOne()
│   ├─ update()
│   └─ delete()
│
├── characters.repository.ts      # CAMADA 3: Banco
│   ├─ save()
│   ├─ findById()
│   ├─ findAll()
│   └─ delete()
│
├── dto/                          # Formato dos dados
│   ├─ create-character.dto.ts
│   ├─ update-character.dto.ts
│   └─ character-response.dto.ts
│
└── entities/                     # Estrutura do banco
    └─ character.entity.ts
```

## 🔄 Fluxo de uma Requisição

### Exemplo: Criar Personagem

```
1️⃣ Cliente faz requisição
   POST /api/characters
   {
     "name": "Gandalf",
     "class": "wizard",
     "level": 20
   }
        │
        ▼
2️⃣ Controller recebe
   @Post()
   create(@Body() dto: CreateCharacterDto)
   │
   ├─ Valida DTO (automático)
   │  ✓ name é string?
   │  ✓ class é válido?
   │  ✓ level entre 1-20?
   │
   └─ Chama Service
        │
        ▼
3️⃣ Service processa
   create(dto)
   │
   ├─ Valida com Core
   │  const validated = CoreValidator.validate(dto)
   │
   ├─ Calcula stats
   │  const stats = CoreCalculator.calculateStats(validated)
   │
   ├─ Salva no banco
   │  const character = await repository.save({...validated, stats})
   │
   ├─ Publica evento
   │  await kafka.publish('character.created', character)
   │
   └─ Retorna
        │
        ▼
4️⃣ Repository salva
   save(data)
   │
   ├─ INSERT INTO characters
   │  (id, name, class, level, stats, created_at)
   │  VALUES (...)
   │
   └─ RETURNING *
        │
        ▼
5️⃣ PostgreSQL
   [Dados salvos]
        │
        ▼
6️⃣ Retorna para Service
   { id: "uuid-123", name: "Gandalf", ... }
        │
        ▼
7️⃣ Service retorna para Controller
        │
        ▼
8️⃣ Controller retorna HTTP
   201 Created
   {
     "id": "uuid-123",
     "name": "Gandalf",
     "class": "wizard",
     "level": 20,
     "stats": { ... },
     "createdAt": "2025-10-05T..."
   }
        │
        ▼
9️⃣ Cliente recebe resposta

🔔 Paralelamente: Kafka propaga evento
   Topic: character.created
   └─ Notification Service → Envia email
   └─ Analytics Service → Registra métrica
   └─ Portrait Service → Gera imagem
```

## 🏗️ Módulos da API

```
rpg-system-api/
│
├── 🎭 Characters Module
│   ├─ Criar personagem
│   ├─ Listar personagens
│   ├─ Buscar por ID
│   ├─ Atualizar personagem
│   └─ Deletar personagem
│
├── ⚔️ Battles Module
│   ├─ Iniciar batalha
│   ├─ Executar turno
│   ├─ Finalizar batalha
│   └─ Histórico de batalhas
│
├── 👤 Users Module
│   ├─ Registro
│   ├─ Login
│   ├─ Perfil
│   └─ Autenticação JWT
│
└── 🔧 Shared Module
    ├─ Database connection
    ├─ Kafka connection
    ├─ Guards (segurança)
    ├─ Interceptors
    └─ Utils
```

## 🔌 Integrações

```
┌──────────────────────────────────────────────────────────┐
│                    rpg-system-api                        │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐    │
│  │ Characters  │  │  Battles    │  │   Users     │    │
│  │   Module    │  │   Module    │  │   Module    │    │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘    │
│         │                │                │             │
└─────────┼────────────────┼────────────────┼─────────────┘
          │                │                │
          ▼                ▼                ▼
┌─────────────────────────────────────────────────────────┐
│              Shared Services                             │
├─────────────────────────────────────────────────────────┤
│  Database  │  Kafka  │  Cache  │  Logger  │  Config   │
└─────┬──────┴────┬────┴────┬────┴────┬─────┴────┬──────┘
      │           │         │         │          │
      ▼           ▼         ▼         ▼          ▼
┌──────────┐ ┌────────┐ ┌──────┐ ┌───────┐ ┌────────┐
│PostgreSQL│ │ Kafka  │ │Redis │ │ File  │ │  Env   │
└──────────┘ └────────┘ └──────┘ └───────┘ └────────┘
```

## 🧩 Dependency Injection

Como os componentes se conectam:

```typescript
// NestJS faz isso automaticamente!

@Module({
  imports: [TypeOrmModule.forFeature([Character])],
  controllers: [CharactersController],
  providers: [
    CharactersService,
    CharactersRepository,
  ],
})
export class CharactersModule {}

// No Controller:
@Controller('characters')
export class CharactersController {
  constructor(
    private readonly service: CharactersService  // ← Injetado!
  ) {}
}

// No Service:
@Injectable()
export class CharactersService {
  constructor(
    private readonly repository: CharactersRepository,  // ← Injetado!
    private readonly kafka: KafkaProducer,              // ← Injetado!
  ) {}
}
```

**Benefício:** Não precisa criar instâncias manualmente!

## 📊 Banco de Dados

### Tabelas Principais

```sql
-- Characters
CREATE TABLE characters (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id),
  name VARCHAR(100) NOT NULL,
  class VARCHAR(50) NOT NULL,
  level INTEGER DEFAULT 1,
  stats JSONB,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

-- Battles
CREATE TABLE battles (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  character1_id UUID REFERENCES characters(id),
  character2_id UUID REFERENCES characters(id),
  winner_id UUID REFERENCES characters(id),
  turns JSONB,
  created_at TIMESTAMP DEFAULT NOW()
);

-- Users
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR(255) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  name VARCHAR(100),
  created_at TIMESTAMP DEFAULT NOW()
);
```

### Relacionamentos

```
users (1) ─────< (N) characters
                        │
                        │
                        ├─< (N) battles (as character1)
                        └─< (N) battles (as character2)
```

## 🔐 Segurança

```
Request
  │
  ├─> AuthGuard (JWT)
  │   ├─ Token válido?
  │   ├─ Usuário existe?
  │   └─ Token expirado?
  │
  ├─> RolesGuard
  │   ├─ Usuário tem permissão?
  │   └─ Role necessária?
  │
  └─> Controller
      └─> Service
```

## 📈 Escalabilidade

### Horizontal (Múltiplas Instâncias)

```
Load Balancer
      │
      ├─> API Instance 1 ─┐
      ├─> API Instance 2 ─┼─> PostgreSQL
      ├─> API Instance 3 ─┤
      └─> API Instance N ─┘
              │
              └─> Kafka
```

### Vertical (Mais Recursos)

```
API Instance
├─ CPU: 2 cores → 4 cores
├─ RAM: 2GB → 4GB
└─ Connections: 100 → 200
```

## 🎯 Resumo das Responsabilidades

| Camada | Responsabilidade | Exemplo |
|--------|------------------|---------|
| **Controller** | HTTP | `@Get()`, `@Post()` |
| **Service** | Lógica | Validar, calcular, orquestrar |
| **Repository** | Dados | `save()`, `findById()` |
| **DTO** | Validação | `@IsString()`, `@Min(1)` |
| **Entity** | Estrutura | Colunas do banco |
| **Module** | Organização | Agrupa tudo relacionado |

## 🚀 Próximos Passos

1. ✅ Entender arquitetura
2. 🔄 Implementar módulo Characters
3. 📋 Adicionar testes
4. 📋 Documentar APIs (Swagger)
5. 📋 Implementar outros módulos

## 💡 Dicas

- **Um arquivo, uma responsabilidade**
- **Nomes claros e descritivos**
- **Testes para cada camada**
- **Documentação inline**
- **Commits pequenos e frequentes**
