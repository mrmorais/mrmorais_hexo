---
title: Testes com Consumer-Driven Contracts
date: 2019-10-22 11:12:12
tags:
---

# Testes com Consumer-Driven Contracts
22 de outubro de 2019


No desenvolvimento de micro-serviços, por exemplo, é importante que haja uma cultura de princípios que facilita o processo de desenvolvimento. Um destes princípios é a **descentralização**: todos os times deve possuir completa liberdade sobre seus serviços, desde o provisionamento de recursos ao deploy independente. Por outro lado, a comunicação entre serviços deve ser coordenada. Um serviço deve responder de forma consistente para que outros serviços possam entendê-lo. Em ambientes em que os serviços são constantemente evoluídos esta coordenação entre as equipes pode ser falível. Como é possível ter a segurança de que os serviços que dependem do meu não ficarão quebrados depois desta alteração? É aí que entram os testes baseados em contratos com consumidores (ou **Consumer-Driven Contracts**).

Este padrão garante a consistência nos relacionamentos entre aplicações ao efetuar testes nas APIs impondo a conformidade com um contrato que é estabelecido.

Vamos considerar as duas aplicações envolvidas:

- **Consumidor** (*consumer*): aplicação que depende e faz utilização de recurso de outro serviço, seja através de HTTP ou mensageria.
- **Provedor** (*provider*): serviço fornecedor de recursos que tem o consumidor como dependente e que atende via API às requisições.

A relação entre estas aplicações é definida pelo contrato, um documento que especifíca as interações existem entre elas: quais requisições o consumidor faz e quais respostas esperadas do provedor. Ou seja, o esquema do contrato é definido com base em exemplos positivos.

O consumidor define como espera ser respondido -- criando o contrato. Quando é feita alterações no código do serviço provedor o teste de validação informará se o contrato foi quebrado ou não com a alteração.

Esta prática possui benefícios práticos:

* Esconde a implementação: os times não precisam conhecer nenhum trecho de código de um serviço de outro time para entender como a resposta é gerada ou como é esperada, tudo fica a cargo de satisfazer o contrato.

- Cria um elo de confiaça: podem ser feitas alterações no provedor sem medo de que outras aplicações sejam prejudicadas (a seguir veremos que, com uma ferramenta de broker de contratos, o provedor pode nem mesmo precisar conhecer quais aplicações dependem dele).

Certamente um dos maiores benefícios é o desacoplamento entre as aplicações uma vez que as interfaces são mais bem definidas.

Neste artigo irei definir esta prática nos termos do framework [Pact](pact.io), uma ferramenta de *contract tests*; provavelmente a mais utilizada para este fim. O ecosistema de Pact inclui também o [Pact Broker](https://docs.pact.io/pact_broker), aplicação para compartilhamento de contratos e resultados de verificações.

Estarei utilizando uma aplicação desenvolvida por mim com base no exemplo do pact-workshop[^repo-pact-workshop] em Javascript, mas Pact possui implementações e guias para diferentes linguagens.

## Funcionamento do Pact

## Testes no lado Consumidor

## Validação no lado Provedor

## Pact Broker

[^repo-pact-workshop]: Repositório no Github: [pact-workshop-js](https://github.com/DiUS/pact-workshop-js)