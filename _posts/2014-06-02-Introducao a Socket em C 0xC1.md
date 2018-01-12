---
layout: post
title: Introducao a Socket em C 0xC1
categories: [c programming]
tags: [socket in C, c coding]
fullview: true
comments: true
---
<p>Vejo gente se perdendo quando vai trabalhar com socket's em C, ou qualquer outro tipo de linguagem o grande lance é que 
não tem como aprender sobre socket's sem saber pelo o menos a base de redes TCP/Ip e por aí vai. A parada é entender redes a 
fundo pra que não precise voltar a estudar e ficar consultando, lógico que consultar é bom sempre ajuda a fixar o conteúdo.
em fim se você está aqui então deve entender o básico de socket é muito importante que tenha a base, vamos aprender sobre Unixes
, ou seja, trabalhar com socket para linux, depois pode vim até um artigo sobre windows, há mais por que o linux Unix ?, é melhor 
meu amigo (pelo simples fato do linux ser foda HAHAH), vai entender com mais clareza e depois vamos partir para o Rindows. Outra 
coisa se você estiver capacitado para fazer comunicações usando socket automaticamente está capaz de criar coisas maliciosas AHAHAHA.
backdoor,trojans,spoof,backconnect,BOTNET e por ae vai.Em fim, vamos parar de conversa e trabalhar()</p>
<h3>Incluindo bibliotecas necessárias em C </h3>
<p>Lembrando não adianta continuar esse lendo se não tiver um conhecimento prévio de TCP/IP, então da uma refrescada na mente
e volte com tudo para nosso aprendizado, então Are you all Right?  Vamos nessa. entendendo melhor os headers que vamos precisar
inserir usar as funções.</p>
<pre class="code">
#include < sys/types >
#include < sys/socket >
</pre>
<p>Esses dois headers ou bibliotecas são as padrões quando falamos de socket(tomada), eles são responsáveis por essas funções aqui</p>
<pre class="code">
socket () bind ()
listen () accept ()
send ()  recv ()
sendto () recvfrom ()
shutdown () connect ().
</pre>
<p>Também temos que inserir o header das funções  htons() e a struct sockaddr_in que vamos ver logo depois</p>
{% highlight c++ %} 
#include < netinet/in.h > 
{% endhighlight  %} 
<p>Vamos agora criar nosso primeiro socket :D.</p>
{% highlight c++ %} 
int Sock;<br>
{% endhighlight  %} 
<p>Criar um socket nada mais é que criar um descritor, uma variável do tipo Int(inteiro).pois a função socket() retorna um número inteiro :D.</p>
<a><h3>Tipos de Socket's</h3></a> 
<pre>
protocolos------------Chamado em C 
|TCP ->               SOCK_STREM           
|UDP ->               SOCK_DGRAM          
|RAW ->               SOCK_RAW
OBS: tem mais lógico(mas eles não são muito cobrados e quase nunca é usado :D)    
</pre>           
--------------------------------
<p>Os tipos de socket que vamos trabalhar  são esses aí, TCP autenticado, UDP sem autentificação e RAW é o fodão para normal, kkk se você ver meu amigo
em algum tipo de aplicação sockets declarado como SOCK_RAW, ai vem merda, já já vai entender o porque disso vamos entender o porque 
de cada um desses socket's.Let's Go Go Go saca só na frase</p>
<a>"Não trate como TCP, quem ti trava como TCP."</a>
<p><a>TCP ---> SOCK_STREM:</a> Com este tipo d Socket podemos ler a gravar em 1 conexão, varias aplicações em redes usam SOCK_STREM, tais como, TELNET, FTP,SSH  e por aí vai... e se você realmente sabe TCP garante a entrega do Pacote o que não ocorre com UDP, em fim você entendeu é simples.<br> 
<a>UDP ---> SOCK_DGRAM:</a> lá vem ele o "largado", vagabundo UDP não quer nem saber, ele não é confiável não se pode confiar nele, no caso
, podemos fazer apenas uma coisa com ele gravar dados ou receber dados :D, lembre-se jamais os dois ao mesmo tempo, como falei ele não é confiável ele não garante a entrega do pacote HEHEHE, ele tem vida própria, Lógico que tem vantagens de se usar UDP né como linhas Voip,skype,hangout(TCP não sei como ficou bom) jogos online e por aí vai.... <br>
<a>RAW --> SOCK_RAW:</a> Tá na hora de Separamos Os Homens Dos Meninos Como eu falei aqui o bicho pega meu parceiro, Tá no ponto!, o socket do tipo 'Deep Web', caixa preta, o lance do 
socket RAW é porque ele vai a fundo na parada, podemos pegar qualquer tipo de informação do pacote, tudo mesmo. Podemos fazer os pacote na Unha. tudo do zero montar. this is power of socket, imagine o poder quando você dominar esse tipo de socket, apenas <a>USER Root</a> pode 
usufruir desse tipo de socket, com o RAW atamos diretamente com a camada de transporte (modelo OSI), podendo manipular tudo. Bem-vindo a 'Deep Socket'  meu parceiro.
</p>
<p>Conclusão Existem mais alguns tipos de socket, SOCK_RDM e SOCK_SEQPACKET, que nunca eu vou usar kkk eu acho né, vamos nós atrelar aos 3 que falei acima. se você quiser ser um GUru da vida, ficar louco esse é o caminho da uma estudada nos outros tipos de socket kkkk. não vamos ocupar nossa mentes com merda né verdade. já vamos entender o do RAW que já começa a loucura total. por hoje é isso não vamos entender em detalhes porque é uma introdução né fera, descanse um pouco e depois volte para nossa saga. até a próxima.<br> 
agradecimento ao Devid Diego :D. </p>