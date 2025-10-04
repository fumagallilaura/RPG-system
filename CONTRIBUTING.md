# Guia de Contribuição

Ficamos felizes com seu interesse em contribuir! Para manter o projeto organizado, por favor siga estas diretrizes.

## 💬 Comunicação

-   Para **bugs**, crie uma [Bug Report Issue](https://github.com/seu-usuario/RPG-system/issues/new?template=bug_report.md).
-   Para **novas ideias**, crie uma [Feature Request Issue](https://github.com/seu-usuario/RPG-system/issues/new?template=feature_request.md).

## 🍴 Fluxo de Trabalho (Workflow)

1.  Faça um **Fork** do repositório.
2.  Crie uma nova branch a partir da `main`: `git checkout -b feature/nome-da-sua-feature`.
3.  Faça seus commits seguindo o padrão [Conventional Commits](https://www.conventionalcommits.org/pt-br/v1.0.0/).
    -   Ex: `feat: adiciona executor de regra de compra por pontos`
    -   Ex: `fix: corrige cálculo de bônus de atributo`
4.  Abra um **Pull Request** para a branch `main` do repositório original.
5.  Aguarde a revisão do código.

## 🔧 Configuração do Ambiente

As instruções estão no [README.md](README.md). Todos os serviços rodam em Docker.

## 🎨 Padrão de Código

-   Usamos ESLint e Prettier para manter o código limpo e consistente.
-   Rode `npm run lint` e `npm run format` antes de commitar.