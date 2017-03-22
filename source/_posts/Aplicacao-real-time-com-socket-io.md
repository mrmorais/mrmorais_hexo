---
title: Aplicação real-time com socket.io
date: 2017-03-22 14:53:39
tags:
---

Neste artigo utilizaremos a biblioteca [socket.io](https://socket.io) para desenvolver uma aplicação multiplayer "real-time". A ideia surgiu quando me foi apresentado um jogo de dados chamado: ["Pig Dice"](https://goo.gl/C5iXdo) como atividade de "Linguagem de Programação I". Mas, enquanto para a matéria eu vou desenvolver a solução em C++, aqui vou criar uma versão web com Node.JS.

Basicamente vou utilizar Express (inicialmente), Socket.io (para comunicar o servidor com os jogadores), React JS (para fazer o front end). Perceba que o servidor não vai persistir informações dos jogadores em um banco de dados. A ideia é apenas armazenas os estados em um array de players. Isso significa que no momento em que o servidor descer, todas as informações de score, users e sockets serão perdidas.

Hoje irei apenas criar uma base simples de conexões, para que nos próximos artigos eu possa implementar realmente a lógica do jogo. Essa base contém somente o socket.io e o servidor web rodando. As features iniciais são, portanto:
- Ao receber nova conexão:
  - Gerar um nome para o usuário e adiciona-lo ao conjunto de usuários.
  - Notificar todos os usuários sobre as mudanças no conjunto.
  - Dizer para o usuário adicionado qual o nome gerado para ele.
- Ao desconectar:
  - Remover o usuário desconectado do conjunto.
  - Notificar todos os usuários sobre as mudanças no conjunto.

Antes de desenvolver estas features, vejamos como funciona o Pig Dice:

## Pig Dice
Um jogo de dados para **duas** pessoas. Cada jogador deve acumular a maior quantidade de pontos possíveis. Os pontos são determinados pelo resultado de cada rodada e cada participante joga uma rodada de cada vez. O primeiro inicia a primeira rodada jogando um dado. Se o número que saiu no dado for 1, o jogador não acumula nenhum ponto e passa a vez para o próximo; se for diferente de 1, ele pode jogar o dado novamente e ir acumulando os pontos na rodada atual. Desde que não caia 1, o jogador pode decidir se para ou continua jogando e acumulando mais pontos (com a possibilidade de tirar 1 e não ganhar nada na rodada). Se escolher parar, os pontos da rodada somam-se aos seus pontos totais. Ganha o jogo o primeiro jogador que fizer 100 ou mais pontos.

Se a explicação não ficou muito clara, recomendo ler sobre o jogo no Wikipédia: [Pig Dice](https://goo.gl/C5iXdo)
