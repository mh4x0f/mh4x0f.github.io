---
layout: post
title: Programando Socket em C 0xC2
categories: [c programming]
tags: [socket in C, c coding]
fullview: true
comments: true
---
<p>continuando nossa saga sobre socket's, o estudo de socket's na minha opinião vale muito a pena o investimento do seu tempo  porque ele é a porta pra você desenvolver as tool malignas do mundo blackhat, vamos estudar algumas dessas ferramentas futuramente.Na última vez que nós aprendemos sobre o headers que ele tem uma função muito importante e declaramos nosso primeiro socket agora, vamos relembrar um pouco.</p>
``` c++
#include < sys/types.h >
#include < sys/sockets.h >
#include < netinet/in.h >
```
<p>A função socket() ela retorna -1 caso for erro então vamos declarar nossa variável novamente e ver como funciona, há lembrando para usar socket em windows é preciso de uma API, no linux não precisamos de nenhuma dll, porque o linux é foda heehheeh
vamos lá declaramos nosso socket e ver alguns detalhes da criação do mesmo :D.</p>
``` c++
int SoCk;
meu_socket = socket (AF_INET, SOCK_STREAM, 0);
```
<p>simples não é, vamos entender o que nós fizemos agora, criamos um variável do tipo Int(inteiro) e logo depois pegando a função socket(){que return -1 no caso de erro :D}. e atribuirmos a família <a>AF_INET</a> o tipo do protocolo <a>SOCK_STREAM (potocolo TCP)</a> e o <a>Zero</a> que indica que vamos fazer uma conexão IP. há é se eu colocar 1 no lugar do 0, agora vai ser uma conexão ICMP, vou deixar uma pequena lista com os tipos. </p>
<pre class="code">
 0 - IP - INTERNET PROTOCOL
  1 - ICMP - INTERNET CONTROL MESSAGE PROTOCOL
  2 - IGMP - INTERNET GROUP MULTICAST PROTOCOL
  3 - GGP - GATEWAY-GATEWAY PROTOCOL
  6 - TCP - TRANSMISSION CONTROL PROTOCOL
  17 - UDP - USER DATAGRAMA PROTOCOL
</pre>
<p>É assim que funciona mano, vamos lá para a família que até esqueci não só tem a AF_INET tem mais algumas como essas ae:</p>
<pre class="code">
+ AF_INET      (ARPA INTERNET PROTOCOLS) - A mais usada
+ AF_UNIX      (UNIX INTERNET PROTOCOLS)
+ AF_ISO       (ISO PROTOCOLS)
+ AF_NS        (XEROX NETWORK SYSTEM PROTOCOLS)
</pre>
<p>Não me interessa essas outras família e tudo mais deixa isso pros psicopatas que querem saber de tudo e nem vou perder meu tempo aprendendo uma coisa que nunca irei usar isso é coisa de doente. kkk Família = tudo que eh da  mesma laia, ou  seja, quando eu
inicio 1 socket especificando o AF_INET, eu estou querendo dizer q to afim d programar
p/ a familia TCP/IP. Comumente voce encontrara' um tal  de
PF_INET ao inves de AF_INET nos sources da galera, por  isso vou  adiantando logo  que
AF_INET se trata de um alias para PF_INET e por esse mesmo motivo, podemos  dizer q de  
certa forma eh mais "correto" usarmos o PF_INET (PF_INET eh mais Hacko! As  pessoas na
rua vão começar a dizer que você realmente sabe o que ta fazendo eheh =).</p>
<p>Vamos avançar nos estudos falar um pouco sobre as estruturas (struct), precisamos usa-las para definimos as funções que citei acima.Vamos ver agora como ta ficando nosso código =).  Existe 1 truque amigo, eh bastante simples, quando vc
nao souber em qual header uma determinada função esta, basta você ir neste diretório:
<br>
 /usr/include
<br>
E procurar a tal função baseando-se pelo diretiva  de  compilação '#include' no  codigo
fonte do programa q  você quer estudar, abra o header e procure pela função, simples =) </p>
<pre>
sys/types.h -> definição de tipos de dados
sys/socket.h -> funções e constantes relativas a sockets
netinet/in.h -> para manipulação de estruturas, dados e endereços da famàlia internet
arpa/inet.h -> definições para funções destinadas a operações com sockets
netdb.h -> permite o uso de funções relativas a hostnames, portas, etc
</pre>
{% highlight c++ %}
#include < sys/types.h >
#include < sys/socket.h >
#include < netinet/in.h >
#include < arpa/inet.h >
#include < netdb.h >

int main()
{
int meu_socket;
meu_socket = socket(AF_INET,SOCK_STREAM,0); // cria meu socket

if(meu_socket == -1)
  printf("Erro ao criar o socket!\n");
else
  printf("Socket criado com sucesso!\n");
return 0;
}
{% endhighlight %}
<p> EHHEH ta ficando Bom, e como disse se nosso descritor nosso socket voltar o valor -1 vai dar uma msg falando onde ta ocorrendo o ERROR poderíamos usar a função Perror mais deixa pra próxima. temos agora que criar a struct com as informações ,Deu até saudade de python como isso é simples, com python. </p>
{% highlight c++ %}
#include < stdio.h > /* para usar printf() */
#include < sys/types.h >
#include < sys/socket.h >
#include < netinet/in.h >
#include < arpa/inet.h >
#include < netdb.h >

int main()
{
int meu_socket;
struct sockaddr_in addr;

meu_socket = socket(AF_INET,SOCK_STREAM,0);

if(meu_socket == -1)
  {
  printf("Erro ao criar o socket!\n");
  return 1;
  }

addr.sin_family      = AF_INET; // 2 Bytes (short)
addr.sin_port        = htons(1234); //2 Bytes (short)
addr.sin_addr.s_addr = INADDR_ANY; //4 bytes

memset(&addr.sin_zero,0,sizeof(addr.sin_zero)); //preenche os 16 bit

if(bind(meu_socket,(struct sockaddr*)&addr,sizeof(addr)) == -1)
  {
  printf("Erro na funcao bind()\n");
  return 1;
{% endhighlight  %}
<p>struct sockaddr_in addr;  Declaração de uma estrutura sockaddr_in responsável por fornecer ao socket as informações sobre a famàlia, endereço e porta que devem ser utilizados para a comunicação. depois é preciso preencher a struct com as informações, como Familia, porta  e IP. a memset vai preencher o restante da estrutura com zero, isso acontece porque IP e a porta são convertido para bytes, a porta com htons e o Ip com INADDR. tudo isso dar 16 bit's. logo depois vem um tratramento com a função <a>BIND</a>, A função bind() é respondível por associar um endereço e porta locais a um socket.</p>
<p>Casa der Erro return -1, agora vem a função listen()coloca o socket em modo de espera definindo um mínimo de 1 conexão pendente que deve ser aguardada.</p>
{% highlight c++ %}
if(listen(meu_socket,1) == -1)
  {
  printf("Erro na funcao listen()\n");
  return 1;
  }
  // Accept() essa função como o nome já demostra ela aceita a primeira conexão que passa pelo Bind(). :D simples
meu_socket = accept(meu_socket,0,0);

if(meu_socket == -1)
  {
  printf("Erro na funcao accept()\n");
  return 1;
  }
{% endhighlight %}
<p>
Agora vejamos a estrutura e as funções utilizadas com mais detalhes:
<br>
struct sockaddr_in addr;
<br>
Uma estrutura do tipo sockaddr_in, definida no header netinet/in.h, contém as informações necessírias para estabelecer uma comunicação entre o computador local e um remoto, baseando-se no seu endereço IP e na sua porta.
Esta estrutura foi criada para simplificar o uso de sockets. A verdade é que funções como bind() e connect(), por exemplo, que utilizam informações sobre porta e endereço para realizarem operações com socket - conectar a um host remoto, configurar localmente um socket, etc -, as obtém de uma estrutura chamada sockaddr.<br>
A estrutura sockaddr é definida da seguinte forma:
</p>
{% highlight c++ %}
strut sockaddr
 {
  unsigned short sa_family;  /* 2 bytes */
  char sa_data[14];          /* 14 bytes */

 }; /* Total = 16 bytes */
{% endhighlight %}
<p>Agora vamos ver como ta ficando nosso código:</p>
{% highlight c++ %}
#include < stdio.h > /* para usar printf() */
#include < sys/types.h >
#include < sys/socket.h >
#include < netinet/in.h >
#include < arpa/inet.h >
#include < netdb.h >

int main()
{
int    meu_socket;
int    sock_cliente;
struct sockaddr_in addr;

meu_socket = socket(AF_INET,SOCK_STREAM,0);

if(meu_socket == -1)
  {
  printf("Erro ao criar o socket!\n");
  return 1;
  }

addr.sin_family      = AF_INET;
addr.sin_port        = htons(1234);
addr.sin_addr.s_addr = INADDR_ANY;

memset(&addr.sin_zero,0,sizeof(addr.sin_zero));

if(bind(meu_socket,(struct sockaddr*)&addr,sizeof(addr)) == -1)
  {
  printf("Erro na funcao bind()\n");
  return 1;
  }

if(listen(meu_socket,1) == -1)
  {
  printf("Erro na funcao listen()\n");
  return 1;
  }

printf("Aguardando conexoes...\n");

sock_cliente = accept(meu_socket,0,0);

if(sock_cliente == -1)
  {
  printf("Erro na funcao accept()\n");
  return 1;
  }

printf("Pedido de conexao feito!\n");

if(send(sock_cliente,"Hi",2,0) == -1)
  {
  printf("Erro ao enviar dados!\n");
  return 1;
  }

close(sock_cliente);         
close(meu_socket);         
return 0;
}
{% endhighlight  %}
<p>A função close() fecha o socket, claro né :D. mas é apenas uma demostração ta faltando agora duas. vamos lá logo abaixo vera que a função send precisa de alguns parâmetros a ser passado como socket, conteudo,tamanho,e a frag. </p>
{% highlight c++ %}
char mensagem[] = "qualquer coisa";
send(sock_cliente,mensagem,strlen(mensagem),0);
{% endhighlight  %}
<p>Nesse caso tou criando uma string e enviando, a frag não mexa normalmente é usada a frag zero são os valores opcionais. Agora para receber os cados utilizamos a função recv(), para pegar os dados enviando pelo cliente e fazer alguma coisa nesse caso vamos aprenas dar um printf.</p>
{% highlight c++ %}

int bytes = recv(sock_cliente,resposta,100,0);

if(bytes == -1) /* Erro */
  printf("Erro ao receber dados!\n");
else
  printf("Recebidos %d bytes.\nDados: %s",bytes,resposta); /* Mostra a resposta do cliente */
{% endhighlight %}
<p>Agora a função Connect() que o nome ja diz tudo né, Utilizamos a função connect() para fazer com que um socket se conecte a um host remoto.</p>
{% highlight c++ %}

if(connect(meu_socket,(struct sockaddr*)&addr,sizeof(addr)) == -1)  
  {
  printf("Erro ao se conectar!\n");
  return 1;
  }
{% endhighlight  %}
<p>A essa função pega aquela struct que definimos e coloca no pacote lógico ela precisa saber o IP e porta e familia para conectar com o cliente. outra coisa quando enviamos algum é preciso inserir um caracter nulo para não da algum tipo de erro usando send(meu_socket,"Alguma coisa\n",13,0);, ou seja precisa qualquer o tamanho exato da string que vai ser enviando. para não perder o costume vamos ver agora um exemplo de como seria enviar uma string para o servidor.</p>
<h3>ServidorChat.c</h3>
{% highlight c++ %}
#include < stdio.h >
#include <sys/types.h >
#include <sys/socket.h >
#include <netinet/in.h >
#include < arpa/inet.h >
#include < netdb.h >

int main()
{
int    meu_socket;
struct sockaddr_in addr_local, addr;

meu_socket = socket(AF_INET,SOCK_STREAM,0);

if(meu_socket == -1)
  {
  printf("Erro ao criar o socket!\n");
  return 1;
  }


/* Preenche a estrutura que serí utilizada com a função bind() */

addr_local.sin_family      = AF_INET;
addr_local.sin_port        = htons(4321);
addr_local.sin_addr.s_addr = INADDR_ANY;

memset(&addr_local.sin_zero,0,sizeof(addr_local.sin_zero));

if(bind(meu_socket,(struct sockaddr*)&addr_local,sizeof(addr_local)) == -1)
  {
  printf("Erro na funcao bind()\n");
  return 1;
  }

/* Estrutura destinada à função connect() */

addr.sin_family      = AF_INET;
addr.sin_port        = htons(1234);
addr.sin_addr.s_addr = inet_addr("127.0.0.1");

memset(&addr.sin_zero,0,sizeof(addr.sin_zero));

printf("Tentando se conectar...\n");

if(connect(meu_socket,(struct sockaddr*)&addr,sizeof(addr)) == -1)  
  {
  printf("Erro ao se conectar!\n");
  return 1;
  }

printf("Conectado!\nEnviando dados...\n");
send(meu_socket,"Alguma coisa\n",13,0);

close(meu_socket);         

return 0;
}
{% endhighlight  %}
<p>Pronto! vamos para uma demostração, vamos criar um simples chat para estabelecer uma comunicação entre cliente e servidor, use o GCC e compile os dois códigos abaixo lembando que você pode alterar a porta que você quiser. é simples calma vamos ainda para a parte maliciosa ehhehe, por em quanto o aprendizado fala mais alto.</p>
<h3>clienteChat.c</h3>
{% highlight c++ %}
#include < stdio.h >
#include < sys/types.h >
#include < sys/socket.h >
#include < netinet/in.h >
#include < arpa/inet.h >
#include < netdb.h >

int main()
{
int    meu_socket;
struct sockaddr_in addr;

meu_socket = socket(AF_INET,SOCK_STREAM,0);

if(meu_socket == -1)
  {
  printf("Erro ao criar o socket!\n");
  return 1;
  }

addr.sin_family      = AF_INET;
addr.sin_port        = htons(1234);
addr.sin_addr.s_addr = inet_addr("127.0.0.1");

memset(&addr.sin_zero,0,sizeof(addr.sin_zero));

printf("Tentando se conectar ao servidor...\n");

if(connect(meu_socket,(struct sockaddr*)&addr,sizeof(addr)) == -1)  
  {
  printf("Erro ao se conectar!\n");
  return 1;
  }

printf("Conectado!\n\n");
int recebidos, enviados;
char mensagem[256];
char resposta[256];

do
  {

  /* O processo inverso é feito aqui. Como o servidor espera uma mensagem inicialmente, o cliente deverí fornecê-la */

  printf("Cliente: ");
  fgets(mensagem,256,stdin);
  mensagem[strlen(mensagem)-1] = '\0';
  enviados = send(meu_socket,mensagem,strlen(mensagem),0);

 /* Após enviar a mensagem, espera-se a resposta do servidor */

  recebidos = recv(meu_socket,resposta,256,0);
  resposta[recebidos] = '\0';
  printf("Servidor: %s\n",resposta);


  }while(recebidos != -1 && enviados != -1);


close(meu_socket);         
return 0;
}
{% endhighlight %}
<a><h3>ClienteChat.c</h3></a>
{% highlight c++ %}
#include < stdio.h >
#include < string.h >
#include < sys/types.h >
#include < sys/socket.h >
#include < netinet/in.h >
#include < arpa/inet.h >
#include < netdb.h >

int main()
{
int    meu_socket;
int    sock_cliente;
struct sockaddr_in addr;

meu_socket = socket(AF_INET,SOCK_STREAM,0);

if(meu_socket == -1)
  {
  printf("Erro ao criar o socket!\n");
  return 1;
  }

addr.sin_family      = AF_INET;
addr.sin_port        = htons(1234);
addr.sin_addr.s_addr = INADDR_ANY;
memset(&addr.sin_zero,0,sizeof(addr.sin_zero));

if(bind(meu_socket,(struct sockaddr*)&addr,sizeof(addr)) == -1)
  {
  printf("Erro na funcao bind()\n");
  return 1;
  }

if(listen(meu_socket,1) == -1)
  {
  printf("Erro na funcao listen()\n");
  return 1;
  }

printf("Aguardando cliente...\n");

sock_cliente = accept(meu_socket,0,0);

if(sock_cliente == -1)
  {
  printf("Erro na funcao accept()\n");
  return 1;
  }

printf("Cliente conectado!\n\n");

int recebidos, enviados; /* Controle de bytes enviados e recebidos */
char mensagem[256];      /* Buffer para envio de mensagens */
char resposta[256];      /* Buffer para receber mensagens  */

do  /* Executa as instruções abaixo ... */
  {
  recebidos = recv(sock_cliente,resposta,256,0);              /* Recebe mensagem do cliente */
  resposta[recebidos] = '\0';                                 /* Finaliza a string com o caractere NULO */
  printf("Cliente: %s\n",resposta); 	       	              /* Mostra a mensagem do cliente */

  printf("Servidor: ");                                       /* Simplesmente informa que deve-se preencher uma mensagem */
  fgets(mensagem,256,stdin);                                  /* Obtém uma mensagem digitada */
  mensagem[strlen(mensagem)-1] = '\0';                        /* Finaliza a string */
  enviados = send(sock_cliente,mensagem,strlen(mensagem),0);  /* Envia a string */

  }while(recebidos != -1 && enviados != -1); /* ... enquanto as funções send() e recv() não retornarem -1 = ERRO */

close(sock_cliente);         
close(meu_socket);         
return 0;
}
{% endhighlight  %}
<p>gretz: Dark_Side </p>
