# Fluxo Principal: Criação de Personagem

Este documento detalha o fluxo completo de criação de um personagem, desde a interação inicial do usuário na interface até o processamento assíncrono pelos microsserviços e a notificação final ao usuário.

## Fase 1: Descoberta e Seleção (Interação do Usuário)

Nesta fase, o usuário escolhe qual tipo de personagem deseja criar. O processo é uma conversa entre o Frontend e a API principal (`service-core-api`), que consulta o banco de dados para apresentar as opções disponíveis.

[Para visualizar o primeiro diagrama no editor, clique aqui](https://mermaid.live/edit#pako:eNq1VF1v0zAU_StXfkAdSkrSJG1qiSG0psDE2NQWkFCkycR3raGxg-1Og6r_Hac0LXTthBDLS3w_fM659yhZkkJxJJQY_LZAWeBAsKlmZS7BPRXTVhSiYtLCe4MaWkOtpEXJT-43vLx6Ay3XdCsK9Aul0WeVONA3YJZ9ZgZz-au2B-yfnh5GovAqm8AzI4zFkhloXSFHYDB3CQbu-EVNldkQssKKW2bxQVWHa05AI5HCOHubnU3gKQxHlxcw3nDvcewmqtNN5D8wyQit0vI37X8iczyCfUSxY9rbIoXz8eU7KFS5hYYW3lEYPBl48IGVldDq5B7bsW39k1OOtZo7VPNi49m14M-Xgl9zyVeNfQZuRDFzLxc4bY9q36QRBB9fZ6MMDsj6_85uSR_B2t1Aa2-H9SZrznM1ZVxpb5e5UNLYv3eceGSqBSfU6gV6pERdsjokyxogJ3aGJeaEuiNn-mtOcrlyd9z3_Umpsrmm1WI6I_SGzY2LFhV3hJv_yzar3Uioz9RCWkI7adRdoxC6JHeE-mGn3Q3iKEq7cZik_Th15e-EhlGnHQb9Xi_sdpKo30mTlUd-rJmDdhr3e3EU9PpxnARJmKx-AonrlpU)

![Diagrama do primeiro fluxo](./images/first-flow.png)

```mermaid
sequenceDiagram
    participant User (Frontend)
    participant API (service-core-api)
    participant Database

    User (Frontend)->>API (service-core-api): GET /sistemas (Pede a lista de jogos)
    activate API (service-core-api)
    API (service-core-api)->>Database: SELECT * FROM Sistemas
    activate Database
    Database-->>API (service-core-api): Retorna lista de Sistemas
    deactivate Database
    API (service-core-api)-->>User (Frontend): JSON com Sistemas (ex: D&D, Vampiro)
    deactivate API (service-core-api)

    User (Frontend)->>API (service-core-api): GET /templates?sistema_id={id_dnd} (Pede as fichas de D&D)
    activate API (service-core-api)
    API (service-core-api)->>Database: SELECT * FROM Templates WHERE sistema_id={id_dnd}
    activate Database
    Database-->>API (service-core-api): Retorna lista de Templates
    deactivate Database
    API (service-core-api)-->>User (Frontend): JSON com Templates (ex: Ficha de Jogador, Ficha de Monstro)
    deactivate API (service-core-api)
```

Ao final desta fase, o usuário selecionou um `template_id` específico, que será a chave para iniciar o processo de criação.

## Fase 2: Requisição, Orquestração e Notificação

Com o `template_id` em mãos, o usuário dispara a criação. A API inicia o processo e responde imediatamente. Em segundo plano, os microsserviços trabalham de forma assíncrona, orquestrados pelo Kafka, desde o processamento inicial até a notificação final de conclusão.

[Para visualizar o segundo diagrama no editor, clique aqui](https://mermaid.live/edit#pako:eNqdVutu2koQfpXp_gGOgHJpSGKdRrLA1UEtFwV6Ih1FqhZ7IavYu-7uGrVFPEzVH30QXuzM2kAdcAIKf8ze5pv5ZubbXRFfBow4RLOvCRM-63G6UDS6F4C_mCrDfR5TYeCzZgrKH5QUhomgcrzBHfehjJuW3Gc1XypWozEv2PeRzh_p8fSdVI8WYWdBG2p0bcEEU9RIVWBoKA2fc58aLgVMsmPHu3rU0BnVuJKtHcRRu7kpdtyB8WgyhbcxU1oKio5oKM9k8N2BFdwTw6I4pIZ94cE9cXCiXq_fE1hv_aS-4UtcfpaVbJuQuEXxxYMBOYfnHHHTlSUNeUCBgsJUcc03vze_JDDwFacgcXbBtVESuMDQaVjPEIptYtQ7XhzoDyfe7RQ_0xGM89HmYqyCzUei35f6w3637_ZGpcNI_xBtp3ej2gsE3zJMrLDOl_YsRwhWSmMKZGYpYM9AnAgwrTPMYjILsUiALZkwEhP1B6qWUYVABMq-jOCJF0htjoBKHjNNW8jmadYOCmqXLsV0LEXAoD_wen136g684dQDKiHRyean4rL-90y9vRmmWdQIbYmIlfSZ1hKhVcQFVS-nEcM8gm81WuD6PosNC6DcRSa3lRJZ6wtlzVeOqH25TlMuLdqpLnXAEwZL0cbyMuMH1XO6_Ys65rQ72x2Jth7l8mk1gsIs0T5VQLVtH0X1lu1TZp-0z8T75HWn8Bd8uB0NoMfmNkYqXaP4LDFSw90_3q2Xx36_4sGX3Xh9dhudDnbXUPt4MMOwwOWsAnBAd16d1Vsp2XKJoGcTzb4xPzF5FxhoGi6Rfjujk9Bg-lFc-gJtCCyGPVGVV5Cf165ji2BF-ZXxuCZBwf1hyyaTPsDE0hACmZOJ19TL53HPRSHIK-3Em-71tTsajD9501HptY6Pcv6hsJjNT9v3thPLwiqNL2Npoxj8O65kEuQupKJVoCJB9rBGwJfCDxONNfPm7ADPkNvMjUKxPRak894DB_JU9CQ4JUk7rw768PnXRZEQFQOnfza_ZaYzEKSVTwOUQMTmGjQDwazep_dBmgsGTCy5vebF1mTWu6lc5e6OzIki2KI7YfjEVtkC3LHZRPqPzFTBq0WUh1Vgxq8fJ6IIg1TJQvGAOEYlrEoivKyoHZKVPY6vowcWsexdFFD1aMld4xl8kP0nZbQ7pmSyeCDOnIYaR0kcINz2_bmfVRgAU12ZCEOc9lVqgzgr8o04rUaz3r6-6HSa7fZF67Jz0ayS78RpNi7qzcb1Zeeyc91qv7u-bK2r5EcK26hf4fhdu9G5ajWvWu1ma_0_KbbLDg)

![Diagrama do segundo fluxo](./images/second-flow.png)

```mermaid
sequenceDiagram
    participant User (Frontend)
    participant API (service-core-api)
    participant Kafka
    participant Worker (service-stats-generator)
    participant Notification Service
    participant Database

    User (Frontend)->>API (service-core-api): POST /personagens (body: { "template_id": "..." })
    activate API (service-core-api)

    note right of API (service-core-api): A API valida a requisição e cria o registro inicial.
    API (service-core-api)->>Database: INSERT INTO Personagens (template_id, status='INICIADO')
    activate Database
    Database-->>API (service-core-api): Retorna o 'personagem_id' criado
    deactivate Database
    
    API (service-core-api)->>Kafka: Publica evento "personagem-iniciado" (com personagem_id e template_id)
    
    note left of User (Frontend): A API responde IMEDIATAMENTE ao usuário.<br/>Não espera o processo terminar.
    API (service-core-api)-->>User (Frontend): 202 Accepted (Criação em progresso)
    deactivate API (service-core-api)

    Kafka-->>Worker (service-stats-generator): Entrega o evento "personagem-iniciado"
    activate Worker (service-stats-generator)

    note right of Worker (service-stats-generator): Worker usa o template_id para buscar as regras.
    Worker (service-stats-generator)->>Database: SELECT * FROM DefinicaoAtributos WHERE template_id={id_template}
    activate Database
    Database-->>Worker (service-stats-generator): Retorna as regras de geração de atributos
    deactivate Database
    
    note over Worker (service-stats-generator): Worker executa as regras e salva os resultados (InstanciaAtributos)
    Worker (service-stats-generator)->>Database: INSERT INTO InstanciaAtributos ...
    
    note over Worker (service-stats-generator): Atualiza o status final do personagem.
    Worker (service-stats-generator)->>Database: UPDATE Personagens SET status='COMPLETO'
    
    note over Worker (service-stats-generator): O personagem está pronto (no escopo do MVP).<br/>Agora, anuncie a conclusão!
    Worker (service-stats-generator)->>Kafka: Publica evento "personagem-pronto" (com personagem_id)
    deactivate Worker (service-stats-generator)
    
    Kafka-->>Notification Service: Entrega o evento "personagem-pronto"
    activate Notification Service
    
    note right of Notification Service: Serviço busca dados adicionais se necessário<br/>e envia a notificação para o usuário.
    Notification Service->>User (Frontend): Notificação (via WebSocket, E-mail, etc.)
    deactivate Notification Service
```

### Conclusão do Fluxo

* A escolha de um **Sistema** e, principalmente, de um **Template** pelo usuário é o que define todo o contexto da criação.
* A arquitetura é **assíncrona**: a API confirma o recebimento da tarefa e o trabalho pesado é feito em segundo plano, garantindo uma experiência de usuário rápida e um sistema escalável.
* O evento final **`personagem-pronto`** é crucial. Ele desacopla o processo de criação da notificação ao usuário e permite que outras funcionalidades (como um gerador de PDF) sejam adicionadas no futuro de forma independente, apenas "ouvindo" o mesmo evento de conclusão.