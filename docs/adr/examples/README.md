# Exemplos de Implementação - ADR 003

Esta pasta contém exemplos completos de código que ilustram os padrões arquiteturais descritos na [ADR 003](../003-arquitetura-api-modular.md).

## 📁 Estrutura

```
examples/
├── README.md (este arquivo)
├── characters.controller.ts    # Controller Layer
├── characters.service.ts       # Service Layer
├── characters.repository.ts    # Repository Layer
├── characters.dto.ts           # DTOs (Input/Output)
├── character.entity.ts         # Entity (Domain Model)
└── observability/
    ├── logging.example.ts      # Logging estruturado
    ├── metrics.example.ts      # Métricas Prometheus
    └── health.example.ts       # Health checks
```

## 🎯 Propósito

Estes exemplos servem como:

1. **Referência de Implementação:** Código completo e funcional seguindo os padrões da ADR
2. **Template:** Base para criar novos módulos
3. **Documentação Viva:** Exemplos que podem ser testados e executados
4. **Separação de Concerns:** Mantém a ADR focada em decisões, não em implementação

## 📝 Como Usar

### Para Criar um Novo Módulo

1. Copie os arquivos de exemplo
2. Renomeie `characters` para o nome do seu módulo (ex: `users`, `battles`)
3. Ajuste as propriedades e lógica de negócio
4. Mantenha a estrutura e padrões

### Para Entender os Padrões

1. Leia a [ADR 003](../003-arquitetura-api-modular.md) primeiro para contexto
2. Estude cada arquivo na ordem:
   - `character.entity.ts` - Modelo de domínio
   - `characters.dto.ts` - Contratos de API
   - `characters.repository.ts` - Acesso a dados
   - `characters.service.ts` - Lógica de negócio
   - `characters.controller.ts` - Camada HTTP
3. Veja como os padrões são aplicados na prática

## ⚠️ Importante

- **Estes são exemplos educacionais:** Podem não estar 100% sincronizados com a implementação real
- **Não copie cegamente:** Entenda os padrões e adapte para seu contexto
- **Mantenha atualizado:** Se a arquitetura mudar, atualize os exemplos

## 🔗 Referências

- [ADR 003: Arquitetura em Camadas](../003-arquitetura-api-modular.md)
- [NestJS Documentation](https://docs.nestjs.com/)
- [Clean Architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
- [Domain-Driven Design](https://martinfowler.com/bliki/DomainDrivenDesign.html)

## 📋 Status dos Exemplos

| Arquivo | Status | Última Atualização |
|---------|--------|-------------------|
| characters.controller.ts | 📋 Pendente | - |
| characters.service.ts | 📋 Pendente | - |
| characters.repository.ts | 📋 Pendente | - |
| characters.dto.ts | 📋 Pendente | - |
| character.entity.ts | 📋 Pendente | - |
| observability/* | 📋 Pendente | - |

> **Nota:** Estes exemplos serão criados conforme a implementação da API avançar.
