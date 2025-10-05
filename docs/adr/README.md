# ADR - Architecture Decision Records

Esta pasta contém os registros das decisões de arquitetura (ADRs) do projeto.

## O que é um ADR?

Um ADR é um documento curto que captura uma decisão de arquitetura importante, junto com seu contexto e consequências. Ele serve para registrar o **"porquê"** de termos escolhido uma determinada abordagem técnica.

## Por que usamos ADRs?

1.  **Clareza e Transparência:** Para que todos no time (atuais e futuros) entendam as razões por trás das escolhas estruturais do projeto.
2.  **Evitar Repetição:** Impede que a mesma discussão técnica aconteça várias vezes.
3.  **Base de Conhecimento:** Cria um histórico valioso da evolução da arquitetura do nosso sistema.

Cada arquivo nesta pasta representa uma única decisão.

## 📚 ADRs Existentes

### Fundamentos
-   [ADR 001: Regras como Dados JSON](001-regras-como-dados-json.md)
-   [ADR 002: Escolha de Multi-Repo](002-escolha-de-multi-repo.md)

### Arquitetura da API
-   [ADR 003: Arquitetura Modular da API](003-arquitetura-api-modular.md) ⭐ **Novo!**
    - Explica a estrutura em camadas (Controller → Service → Repository)
    - Analogias simples para iniciantes
    - Guia completo de organização de código

## 📖 Documentação Complementar

- [Arquitetura da API - Guia Visual](../arquitetura-api-visual.md) - Diagramas e exemplos práticos

## Criando um Novo ADR

Para registrar uma nova decisão de arquitetura, siga os passos abaixo:

1.  Copie o arquivo `TEMPLATE.md` para um novo arquivo dentro desta pasta.
2.  Renomeie o novo arquivo seguindo o padrão `XXX-titulo-curto-da-decisao.md`, onde `XXX` é o próximo número sequencial (ex: `004-escolha-do-orm.md`).
3.  Preencha todas as seções do template no novo arquivo. Comece com o status "Proposto".
4.  Submeta o novo ADR como um Pull Request. A discussão e aprovação da decisão acontecerá no próprio PR.
5.  Após a aprovação e o merge, o status do ADR é alterado para "Aceito".

## 💡 Dicas para Escrever ADRs

- **Use analogias simples** (restaurante, casa, etc) - ajuda quem não é da área
- **Explique "por que"** além de "como"
- **Inclua exemplos de código** quando relevante
- **Liste alternativas consideradas** - mostra que pensou bem
- **Seja honesto sobre trade-offs** - toda decisão tem prós e contras
- **Adicione glossário** se usar termos técnicos