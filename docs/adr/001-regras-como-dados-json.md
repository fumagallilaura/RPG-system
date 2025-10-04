# ADR 001: Uso de JSON para Descrever Regras de Geração

-   **Status:** Aceito
-   **Data:** 2025-10-04

## Contexto

O sistema precisa suportar múltiplos sistemas de RPG (D&D, Vampiro, etc.), cada um com regras de criação de atributos fundamentalmente diferentes. Implementar essa lógica diretamente no código dos serviços geraria um acoplamento forte e um código complexo e difícil de manter.

## Decisão

As regras de geração de atributos (e outras no futuro) serão armazenadas no banco de dados em um campo `JSONB`. O microsserviço responsável (`service-stats-generator`) será um "executor de regras" genérico. Ele lê a descrição da regra em JSON e executa a lógica correspondente.

**Exemplo (D&D):**
`{ "metodo": "rolagem_dados", "parametros": { "quantidade_dados": 4, ... } }`

## Consequências

-   **Positivas:**
    -   **Alta Escalabilidade:** Adicionar um novo sistema de RPG se torna uma operação de inserção de dados no banco, sem a necessidade de deploy de código.
    -   **Desacoplamento:** O serviço de geração não conhece as regras específicas de nenhum jogo.
    -   **Clareza:** As regras de um sistema ficam centralizadas e explícitas no `Template`.
-   **Negativas:**
    -   O serviço executor precisa ser capaz de interpretar diferentes "métodos" definidos no JSON.
    -   Regras muito complexas podem ser difíceis de modelar em JSON.