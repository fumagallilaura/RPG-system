# RPG System - A Forja de Heróis

Uma plataforma assíncrona e extensível para geração de personagens de RPG sob demanda, construída com uma arquitetura de microsserviços orientada a eventos e com forte viés em arquitetura de software e soluções.

## ✨ Features Principais

-   **Multi-Sistema:** Suporte para D&D, Vampiro a Máscara, 3D&T e qualquer outro sistema através de um modelo de regras dinâmico.
-   **Geração Assíncrona:** Solicite um personagem e seja notificado quando a ficha completa, incluindo stats, backstory e retrato, estiver pronta.
-   **Extensível:** Adicionar um novo sistema ou uma nova regra de geração é uma questão de configuração, não de código.

## 🚀 Tecnologias

-   **Backend:** TypeScript, Node.js, NestJS
-   **Mensageria:** Apache Kafka
-   **Banco de Dados:** PostgreSQL
-   **Containerização:** Docker

## 🏁 Começando (Getting Started)

1.  Clone o repositório: `git clone git@github.com:fumagallilaura/RPG-system.git`
2.  Navegue até a pasta: `cd RPG-system`
3.  Suba o ambiente: `docker-compose up -d`
4.  A API principal estará disponível em `http://localhost:3000`.

## 🤝 Como Contribuir

Leia nosso [Guia de Contribuição](CONTRIBUTING.md) para entender nosso fluxo de trabalho e como você pode nos ajudar.

## 📄 Documentação

-   [Modelo de Dados](docs/data-model.md)
-   [Decisões de Arquitetura (ADRs)](docs/architecture-decision-record/)