---
layout: post
title: Entendendo Buffer Overflow - 0x02
categories: [Exploit Developer]
tags: [Buffer overflow]
fullview: true
comments: true
---
Seguindo com artigos x01, vamos falar sobre buffers, fizemos um breve introdução sobre o assunto Buffer no outro artigos, nesse veremos os tipos de buffers.
<h3>Buffer - o que é?</h3>
<p>(wiki) - Em Segurança computacional e programação um transbordamento de dados ou estouro de buffer (do inglês buffer overflow ou buffer overrun) é uma anomalia onde um programa, ao escrever dados em um buffer, ultrapassa os limites do buffer e sobrescreve a memória adjacente. Esse é um caso especial de violação de segurança de memória. Estouros de buffer podem ser disparados por entradas que são projetadas para executar código, ou alterar o modo como o programa funciona. Isso pode resultar em comportamento errado do programa, incluindo erros de acesso à memória, resultados incorretos, parada total do sistema, ou uma brecha num sistema de segurança. Portanto, eles são a base de muitas vulnerabilidade de software e pode ser explorados maliciosamente.</p>
<h3>Tipos de Buffers</h3>
<p>Os tipos mais comuns de ataques baseados em buffer overflow são:
? Stack-based buffer overflow
? Heap-based buffer overflow
? Return-to-libc attack</p>

<p>Calma, iremos abordar cada um desses detalhadamente, mas para compreender melhor como funciona o ataque em si, já sabemos que buffer é um espaço revesado, e overflow transbordo desses dados, mas como funciona a memoria sabemos disso ? é uma das partes fundamentais entender como funciona, let's go. </p>
<h3>Sequência de instruções</h3>
<p>Qualquer programa compilado (e isto inclui interpretadores de linguagens) possui uma
forma estruturada em código de máquina denotando uma seqüência de instruções que serão
executadas pelo processador. Tal seqüência é carregada na memória quando um programa é
chamado à execução.<br>
A estrutura de um programa na memória se divide em
quatro partes: texto, dados, stack (pilha) e heap.
<pre>
                  +-------------------+
                   |                   |   <----+
                   |       texto       )|       |
                   |                   |        |
                   +       dados       +        |
                   |         |         |        |
                   |       heap        |        |
                   |                   |        |
                  <                     >       |
                   |                   |        |
                   |         ^         |        |
                   |         |         |        |
                   +         |         +        |
                   |                   |        |
                   |       pilhas      |        |
                   |                   |        |      +-------------------------+
                   +-------------------+        +-----<  Classical Memory Layout |
</pre>
</p>
<br>
<p>
<a>texto</a>>> contém as instruções do programa
propriamente ditas e dados somente-leitura. Essa região é
geralmente somente-leitura, para evitar códigos mutáveis. Uma
tentativa de escrita nessa área resulta em uma falha de
segmentação.<br>
<a>dados</a>>> contém todas as variáveis globais e
estáticas do programa.<br>
<a>heap</a>>> serve para a alocação dinâmica de
variáveis, através de instruções do tipo malloc();.<br>
<a>pilha</a>>> é a região  Trata-se de um bloco de
memória onde são armazenadas informações vitais para subrotinas: variáveis locais, parâmetros e
os valores de retorno.</br>
<p>Algumas definições básicas antes de começar: Um buffer é simplesmente uma contígua
bloco de memória de computador que contém várias instâncias dos mesmos dados
digita. Programadores C associa normalmente com as matrizes de buffer de texto. A maioria
comumente, arrays de caracteres. Arrays, como todas as variáveis &#8203;&#8203;em C, pode ser
declarou estático ou dinâmico. Variáveis &#8203;&#8203;estáticas são alocados em carga
tempo para o segmento de dados. As variáveis &#8203;&#8203;dinâmicas são alocados em tempo de execução em
a pilha. Para transbordar é a fluir, ou preencher por cima, transborda, ou limites.</p><br>
<p>No linux a coisa já começa a ficar diferente vou focar em linux até porque linux é meu postor,tenho que seguir os mandamentos software livre, em breve vamos falar um pouco sobre assembly nada dimais apenas uma leve introdução aos registradores que varia de arquitetura, vamos entender também as proteções tanto do  linux tanto o windows, mas para o windows estou fazendo um paper falando sobre DEP , que é um controlador de violação de memória,em fim, entederá melhor como fazer um bypass(passar), pelos AV usando essa função.No linux a coisa muda meu amigo, quer dizer a estrutura do linux fica assim:  </p><br>
<h3>Divisão de Memória no linux</h3>
<p>O modelo usual de memoria depende muito  do sistema, me utilizarei como base inicial o
modelo no "Linux/i186" apenas para  demonstracao, apesar dos modelos usuais de memoria
serem diferentes para cada sistema, o ponto chave e' o mesmo, Heap  é um local, stack  
é outro.</p></br>
<pre >

     +-----------+
     |  .DATA    |  <-- Regiao onde ocorre o Stack overflow
     +-----------+
     |  .BSS     |  <-- Regiao onde ocorre o Heap Overflow
     +-----------+
     |  .TEXT    |  <-- Instrucoes em Assembly (Written in Assembly)
     +-----------+

</pre>

<p> A região de dados contém dados inicializados e não inicializadas. Estático
variáveis &#8203;&#8203;são armazenados nesta região. A região de dados corresponde à
seções-BSS dados do arquivo executável. O seu tamanho pode ser alterado com o
brk (2) chamada de sistema. Se a expansão dos dados de BSS ou a pilha de utilizador
esgotar a memória disponível, o processo é bloqueado e é remarcado para
correr novamente com um espaço de memória maior. Nova memória é adicionado entre os dados
e empilhar segmentos.</p><br>
<p>Alguns de nós podem pensar que, apesar de um buffer overflow é uma má prática de programação, mas isso é uma variável não utilizada na pilha, então por que há tanto barulho em torno dele? O que é o buffer de dano invadida pode causar para a aplicação?

Bem, se em uma linha temos que resumir a resposta a estas perguntas, então seria:

Estouros de buffer, se não detectada, pode fazer com que seu programa para falhar ou produzir resultados inesperados. :D até a próximo. </p>
