# ADR - Architecture Decision Records

Esta pasta contém os registros das decisões de arquitetura (ADRs) do projeto.

## O que é um ADR?

Um ADR é um documento curto que captura uma decisão de arquitetura importante, junto com seu contexto e consequências. Ele serve para registrar o **"porquê"** de termos escolhido uma determinada abordagem técnica.

## Por que usamos ADRs?

1.  **Clareza e Transparência:** Para que todos no time (atuais e futuros) entendam as razões por trás das escolhas estruturais do projeto.
2.  **Evitar Repetição:** Impede que a mesma discussão técnica aconteça várias vezes.
3.  **Base de Conhecimento:** Cria um histórico valioso da evolução da arquitetura do nosso sistema.

Cada arquivo nesta pasta representa uma única decisão.

## Criando um Novo ADR

Para registrar uma nova decisão de arquitetura, siga os passos abaixo:

1.  Copie o arquivo `TEMPLATE.md` para um novo arquivo dentro desta pasta.
2.  Renomeie o novo arquivo seguindo o padrão `XXX-titulo-curto-da-decisao.md`, onde `XXX` é o próximo número sequencial (ex: `003-adocao-de-websockets.md`).
3.  Preencha todas as seções do template no novo arquivo. Comece com o status "Proposto".
4.  Submeta o novo ADR como um Pull Request. A discussão e aprovação da decisão acontecerá no próprio PR.
5.  Após a aprovação e o merge, o status do ADR é alterado para "Aceito".