---
title: Testes com Consumer-Driven Contracts
date: 2019-10-22 11:12:12
tags:
---

# Testes com Consumer-Driven Contracts
22 de outubro de 2019


No desenvolvimento de micro-serviços, por exemplo, é importante a existência de uma cultura de princípios guiando o processo de desenvolvimento. Um destes princípios é a **descentralização**: todos os times deve possuir controle e liberdade sobre seus serviços (desde o provisionamento de recursos ao deploy independente). Um ponto importante para garantir descentralização é a coordenação entre serviços, que devem ser o mais desacoplados possível. Um serviço deve responder de forma consistente para que outros serviços possam utilizá-lo apropriadamente, mas em ambientes de constantes modificações esta coordenação entre equipes pode ser falível e demandar testes de integração complexos de e2e (*end-to-end*).

<details>
<summary>Definição de testes e2e</summary>
Testes e2e (de ponta a ponta) são testes de mais alto nível e que exigem ambiente com todo o sistema (ou uma parte) em execução, eficaz em testar lógicas de negócios pensando no usuário.<br/>
</details><br/>

Como é possível ter a segurança de que serviços inter-dependentes não sejam negativamente impactados com mudanças? Esta é a proposta dos testes baseados em contratos com consumidores (ou **Consumer-Driven Contracts**). Este padrão garante a consistência nos relacionamentos entre aplicações ao efetuar testes nas APIs impondo a conformidade com um contrato que é estabelecido pelas expectativas das aplicações que consomem as APIs.

Logo, temos dois tipos de aplicações envolvidas:

- **Consumidor** (*consumer*): aplicação que depende e faz utilização de recurso de outro serviço, seja através de HTTP ou mensageria.
- **Provedor** (*provider*): serviço fornecedor de recursos que tem o consumidor como dependente e que atende via API às requisições.

A relação entre estas aplicações é definida por uma forma de contrato, um documento que especifíca as interações existentes: quais requisições o consumidor faz e quais respostas são esperadas do provedor para essas requisições. Ou seja, o esquema do contrato é definido com base em exemplos positivos.

O consumidor define como espera ser respondido (momento de criação do contrato.) Quando são feitas alterações no código do serviço provedor, o teste de validação informa se o contrato foi quebrado em decorrencia da alteração, interropendo o progresso da build na pipeline de deploy, por exemplo.

Esta prática possui benefícios práticos:

* Esconde a implementação: os times não precisam conhecer nenhum trecho de código de um serviço de outro time para entender como a resposta é gerada ou como é esperada, tudo fica a cargo de satisfazer o contrato.

- Cria um elo de confiaça: podem ser feitas alterações no provedor sem medo de que outras aplicações sejam prejudicadas (a seguir veremos que, com uma ferramenta de broker de contratos, o provedor nem mesmo precisa conhecer quais aplicações dependem dele).

Certamente um dos maiores benefícios é o desacoplamento entre as aplicações uma vez que as interfaces são mais bem definidas.

Neste artigo irei definir esta prática nos termos do framework [Pact](pact.io), uma ferramenta de *contract tests*; provavelmente a mais utilizada para este fim. O ecosistema de Pact inclui também o [Pact Broker](https://docs.pact.io/pact_broker), aplicação para compartilhamento de contratos e resultados de verificações.

Estarei utilizando uma aplicação desenvolvida por mim com base no exemplo do pact-workshop[^repo-pact-workshop] em Javascript, mas Pact possui implementações e guias para diferentes linguagens.

## Funcionamento do Pact

Pact atua como *mock* nos dois lados da interação: para o consumer ele faz o papel do provider e o contrário para o provider. O ponto de partida é a especificação no lado consumer. O Pact intercepta as requisições que iriam para o provider respondendo com o que o consumer define no Pact test como esperado, neste momento os testes unitários do consumer também são realizados para garantir que com a resposta esperada do provider ele saberá operar como esperado também.

Ao final dos testes no lado consumer, se todos os teste passam, é gerado um arquivo `.json` com um conjunto de interações (especificações de requisição e resposta esperada). Este arquivo é, por assim dizer, o pacto (contrato) do relacionamento, o provider precisará somente dele para validar se está de acordo com as especificações do consumer. O pacto deve ser distribuído para que o provider tenha acesso. Existem várias formas de fazer esta distribuição: transferir via diretório compartilhado, repositório git ou, o mais recomendado, um Pact Broker (que é abordado na sessão [Pact Broker](#pact-broker)).

A figura abaixo dá uma visão geral do funcionamento do Pact.

<br/>

![](https://blobscdn.gitbook.com/v0/b/gitbook-28427.appspot.com/o/assets%2F-LC2AYrI9MJa-_aAjE1u%2F-LN88wE6mKsXwlpUIcxu%2F-LN88wz8GOdj0witaWgd%2Fpact-test-and-verify.png?generation=1537751898934036&alt=media)
<center><small>Panorama geral dos componentes sob teste em Pact. Fonte: https://docs.pact.io/getting_started/how_pact_works</small></center><br/>

## Aplicação demonstrativa

A aplicação exemplo no lado consumer passa no corpo da requisição uma lista de itens (descrições, preços e quantidades) para o provider, que deve, então, calcular o valor total de uma compra com tais itens.

Este é o código-fonte do consumer:

```js
const request = require('superagent')

const API_HOST = process.env.API_HOST || 'http://localhost';
const API_PORT = process.env.API_PORT || 3000;
const apiUri = `${API_HOST}:${API_PORT}`;

const checkoutOrder = items => {
    return request
        .post(`${apiUri}/checkout`)
        .send({
            items
        })
        .then( res => {
            return {
                totalAmount: res.body.totalAmount
            };
        })
};

module.exports = {
    checkoutOrder
}
```

### Testes no lado *Consumer*

O teste vai ser definido utilizando o framework Mocha com Chai. O Pact entra como definição de mock do provider. É feita a importanção e a definição do Pact em que são especificados nomes, portas e arquivos em que serão realizados outputs de logs e do pacto.

<center>consumerPact.spec.js</center>

```js
const Pact = require('@pact-foundation/pact').Pact;

const provider = new Pact({
    consumer: 'My consumer',
    provider: 'My provider',
    port: 3000,
    log: path.resolve(process.cwd(), 'logs', 'pact.log'),
    dir: path.resolve(process.cwd(), 'pacts'),
    logLevel: 'warn',
    spec: 2
});
```

No corpo do teste esta definida a interação "a request for total amount" que define o formato da requisição (`withRequest`) e a resposta que o mock criado pelo Pact irá dar ao consumer (`willRespondWith`). Em seguida é feita a validação da resposta, verificando se o `totalAmount` retornado pelo mock é igual a um valor esperado (156.5).

Neste momento convém testar unitáriamente o próprio consumer. Neste exemplo a função `checkoutOrder` retorna simplesmente o valor obtido do provider mas se houvesse, por exemplo, o cálculo de preço médio dos produtos feito pelo consumer com o retorno do provider teriamos que fazer asserts nesse valor. O motivo é simples, não é interessante gerar um contrato em que a lógica das funcionalidades do consumer não são satisfeitas.

<center>consumerPact.spec.js</center>

```js
describe('Consumer Pact', () => {
    before(() => {
        return provider.setup();
    });

    describe('when a call to the Provider is made', () => {
        describe('and passing a valid items set', ()=> {
            before(() => {
                return provider.addInteraction({
                    uponReceiving: 'a request for total amount',
                    withRequest: {
                        method: 'POST',
                        path: '/checkout',
                        body: { items },
                        headers: {
                            'Content-Type': 'application/json'
                        }
                    },
                    willRespondWith: {
                        status: 200,
                        headers: {
                            'Content-type': 'application/json; charset=utf-8',
                        },
                        body: {
                            totalAmount: 156.5
                        }
                    }
                });
            });

            it('can process the JSON payload from provider', () => {
                const response = checkoutOrder(items);

                return expect(response).to.eventually.have.property('totalAmount', 156.5)
            });

            it('should validate and create a contract', () => {
                return provider.verify();
            })
        });
    });

    after(() => {
        provider.finalize();
    })
});
```

Quando finalizada a execução do teste, o Pact cria, no diretório definido anteriormente, o arquivo com as especificações. Este arquivo de pacto deve ser distribuído para o provider. Este projeto utiliza [Pactflow](https://pactflow.io/), ferramenta online de Pact Broker para distribuição de pactos. É possível, também, executar Pact Broker em um servidor próprio.

<center>my_consumer-my_provider.json</center>

```
{
  "consumer": {
    "name": "My consumer"
  },
  "provider": {
    "name": "My provider"
  },
  "interactions": [
    {
      "description": "a request for total amount",
      "request": {
        "method": "POST",
        "path": "/checkout",
        "headers": {
          "Content-Type": "application/json"
        },
        "body": {
          "items": [
            {
              "description": "Antique typewriter",
              "price": 49,
              "quantity": 1
            },
            {
              "description": "Christmas decorations",
              "price": 15,
              "quantity": 5
            },
            {
              "description": "Notepad",
              "price": 16.25,
              "quantity": 2
            }
          ]
        }
      },
      "response": {
        "status": 200,
        "headers": {
          "Content-type": "application/json; charset=utf-8"
        },
        "body": {
          "totalAmount": 156.5
        }
      }
    }
  ],
  "metadata": {
    "pactSpecification": {
      "version": "2.0.0"
    }
  }
}
```

### Validação no lado *Provider*

### Pact Broker

## Conteúdos relacionados

Grande parte do conteúdo deste artigo se baseia conceitualmente na publicação em vídeo "The Principles of Microservices" de O'Reilly Media apresentado por Sam Newman e no livro "Building Microservices" também do Newman (uma leitura recomendada).

Imagens e conceitos do framework Pact são do [Pact](pact.io).

[^repo-pact-workshop]: Repositório no Github: [pact-workshop-js](https://github.com/DiUS/pact-workshop-js)