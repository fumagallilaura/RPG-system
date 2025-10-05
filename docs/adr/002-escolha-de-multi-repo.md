# ADR 002: Adoção da Estratégia Multi-Repositório (Multi-Repo)

-   **Status:** Aceito
-   **Data:** 2025-10-04

## Contexto

O projeto RPG System é composto por múltiplos microsserviços (API, Gerador de Stats, Notificações, etc.). Precisamos decidir como o código-fonte desses serviços será organizado e versionado. As duas abordagens mais comuns no mercado são:

1.  **Monorepo (Repositório Único):** Todos os serviços vivem dentro de um único e grande repositório Git. É como ter uma **Grande Biblioteca Central** onde todos os livros (serviços) estão no mesmo prédio.
2.  **Multi-repo (Múltiplos Repositórios):** Cada serviço vive em seu próprio repositório Git, completamente independente. É como ter várias **Bibliotecas Especializadas de Bairro**, cada uma com seu próprio prédio e suas próprias regras.

A decisão impacta diretamente o fluxo de trabalho dos desenvolvedores, a automação (CI/CD) e como o código compartilhado é gerenciado.

## Decisão

**Decidimos adotar a abordagem Multi-Repositório.**

Cada microsserviço terá seu próprio repositório no GitHub. Esta abordagem foi escolhida por sua clareza conceitual (um repositório = um serviço), o que facilita o entendimento da posse e do escopo de cada parte do sistema.

Para resolver o desafio de compartilhar código comum (como tipos de dados e interfaces TypeScript) entre os diferentes serviços, a seguinte estrutura será adotada:

1.  **Repositório de Código Compartilhado:** Será criado um repositório chamado `rpg-system-core`. Ele não conterá lógica de serviço, apenas o código que precisa ser consistente em todo o ecossistema (tipos, interfaces, classes de erro, etc.). Este repositório será publicado como um pacote NPM no GitHub Packages.

2.  **Estrutura Proposta dos Repositórios:**
    * `RPG-system`: (Este repositório) Servirá como o ponto central para toda a documentação de arquitetura e decisões.
    * `rpg-system-core`: O pacote NPM compartilhado.
    * `rpg-system-api`: O microsserviço da API principal.
    * `rpg-system-stats-generator`: O microsserviço worker para gerar atributos.
    * `rpg-system-notification-service`: O microsserviço para enviar notificações.
    * `rpg-system-infra`: (Opcional, mas recomendado) Um repositório para orquestrar o ambiente de desenvolvimento local com Docker Compose.

## Consequências

### Positivas

-   **Clareza e Isolamento:** Cada serviço é completamente independente, com seu próprio histórico de commits e seu próprio ciclo de deploy. O modelo mental é simples.
-   **Repositórios Leves:** As operações do Git (clone, pull) são rápidas em cada repositório.
-   **Controle de Acesso:** É possível dar a um desenvolvedor ou time permissão para acessar apenas os repositórios nos quais eles trabalham.

### Negativas (e Como Mitigaremos)

-   **Gestão de Dependências:** A maior complexidade passa a ser o gerenciamento do pacote `rpg-system-core`. Uma mudança neste pacote exigirá a atualização da dependência em todos os serviços que o utilizam, gerando uma cascata de atualizações. Mitigação: Usaremos versionamento semântico (SemVer) e automação para gerenciar essas atualizações.
-   **Configuração do Ambiente Local:** O ambiente de desenvolvimento se torna mais complexo, exigindo o clone de múltiplos repositórios. Mitigação: Usaremos um repositório `rpg-system-infra` com um `docker-compose.yml` bem documentado para orquestrar a subida do ambiente completo.
-   **Refatorações Globais:** Mudar algo que afeta múltiplos serviços (ex: renomear um campo em um evento do Kafka) se torna uma tarefa mais complexa, exigindo Pull Requests em vários repositórios. Mitigação: O impacto dessas mudanças será analisado e planejado com cuidado.