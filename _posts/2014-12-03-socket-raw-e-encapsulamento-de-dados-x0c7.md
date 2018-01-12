---
layout: post
title: Socket Raw e encapsulamento de Dados 0xc7
categories: [c programming]
tags: [socket raw, C,encapsulamento de dados, OSI]
fullview: true
comments: true
---
<p> Chegou a hora de conhecer os famosos socket raw , já tava até demorando de escrever sobre. Nesse artigo vamos entrar  no mundo do tráfico, keep me calm, não vamos virar traficante nem de drogas e nem de humanos, kkkk vamos entender como funciona, cada detalhe, cada fração de segundos  (não é pra tanto!!), em fim, vamos atuar na canada TCP  pilha de protocolos TCP/IP, quem sabe o UDP em breve, praticamente vamos entrar na AFASC e entender tudo que acontece por lá, tráfico de  pacotes, camadas,datagramas etc...  Além disso, vamos atuar no projeto semente onde veremos alguns conceitos de rede aliando aos conhecimento já adquiridos em C, you ready ? we go...</p>
<h3>Introdução a Socket Raw</h3>
<p>A ideia ou conceito básico de se usar o socket de baixo nível (low level), é basicamente enviar um pacote por vez, tudo que tem direito é como um (Xtudo), não vou fazer propaganda para Mcdonald's ehehhheheh, usando os socket raw podemos preencher o nosso pacote da maneira que desejar, cabeçalhos de protocolos e ou invés de deixar o kernel fazer tudo, no Unix podemos usar dois tipos de socket's o SOCK_PACKET que já vimos em artigos passados, e o SOCK_RAW que é nosso modo  Hardcore na verdade. sem falar que com eles podemos fazer spoof de IP, attack dos,sniffers em texto plano (veremos e breve),etc... irei falar mais sobre eles na conclusão. mas antes de começamos a por a mão na massa, vamos entender um pouco sobre o modelo das camadas OSI Open  Systems Interconnection     -> Sistemas  de   interconexão  aberta;</p>
<h3>OSI Open  Systems Interconnection</h3>
<p>O Modelo OSI (acrônimo do inglês Open System Interconnection) é um modelo de rede de computador referência da ISO dividido em camadas de funções, criado em 1971 e formalizado em 1983, com objetivo de ser um padrão.(Wiki) Esse padrão foi constituido para funcionar dessa forma com 7 camadas.,vamos lá cada uma delas;</p>
<img src="/images/posts/modelo-osi2.jpeg">
<h3>Camada  aplicação 7 </h3>
<p> Essa é a camada que resume nosso tutorial, a camada responsável por criar pacotes, tudo relacionado a solicitação de conexão. exemplos browers, firefox,chrome,iexplorer(feito para instalar os outros navegadores) . quando você faz um request para um servidor qualquer você está criando um pacote com header e tudo mais veremos isso em breve, Nela é que atuam o DNS, o Telnet, o FTP, etc.</p>
<h3>Camada de Apresentação 6</h3>
<p>Essa camada prepara os dados que serão enviados, é como entregar para o correio enviar seu pacote. ela converte os dados  para um formato compatível para o procedimento de transporte.por outras palavras essa camada já é inserida na camada TCP/IP  e ela é a camada responsável por fazer com que duas redes diferentes (por exemplo, uma TCP/IP e outra IPX/SPX) se comuniquem, ?traduzindo? os dados no processo de comunicação. </p>
<h3>Camada de Sessão 5</h3>
<p>Camada responsável por estabelecer uma conexão entre  dois hosts, a camada de seção têm que se preocupar com a sincronização entre hosts, para que a sessão aberta entre eles se mantenha funcionando. novamente varemos isso em C :D, na parte do header do pacote, aguarde... Esta camada também é responsável pelo CONTROLE da comunicação entre as aplicações  que estão em execução nas maquinas. No TCP/IP esta camada já é inserida nas aplicações de rede.</p>
<h3>Camada de Transporte 4</h3>
<p> A camada de transporte é responsável pela qualidade na entrega/recebimento dos dados. Após os dados já endereçados virem da camada 3, é hora de começar o transporte dos mesmos. definimos  como o  nosso pacote vai ser  'guiado' pelo protocolo IP (Protocolo responsável  pelo  roteamento de  pacotes na  internet). Uma  determinada  aplicação  (Software's) poderia  utilizar  o  UDP  ou  poderia utilizar o famoso TCP.
<h3>Camada de Rede 3</h3>
<p>A camada 3 é responsável pelo tráfego no processo de internet. Nesta camada  que é  inserido ao header TCP ou UDP do pacote, um header com o protocolo IP que vai ser o encarregado  de transportar os dados. costumamos dizer que essa camada é dos sonhos pois nela podemos fazer coisas um pouco malignas na rede, só mais uma informação  ela endereça os dados pelas redes, e gerencia suas tabelas de roteamento.:D tem muita coisa a falar dessa camada como classes de IPs, Broadcast da uma olhada ai na net. não vamos fugir do assunto;</p>
<h3>Camada de Enlace 2</h3>
<p> Camada   responsável  pela  comunicação  entre  duas  interfaces  de rede em uma LAN (Local Area Network). O trabalho  de receber e entregar o pacote para uma determinada interface de rede, Após a camada física ter formatado os dados de maneira que a camada de enlace os entenda, inicia-se a segunda parte do processo. Um aspecto interessante é que a camada de enlace já entende um endereço, o endereço físico (MAC Address ? Media Access Control ou Controle de acesso a mídia)DNS spoof,DNS Posion, MTIM, etc... :D </p>
<h3>Camada Física</h3>
<p>É  a camada   onde se inicia o todo processo. a camada física identifica como 0 sinal elétrico com ?5 volts e 1 como sinal elétrico com +5 volts. Diz respeito a parte fisica da  rede, quando digo  física pode pensar em eth0 (Ethernet) e tals. em fim, não irei abortar nada dessa camada por enquanto :D. </p>
<h3>Encapsulamento de dados</h3>
<p>Agora que já sabemos como funciona as camadas do modelo OSI, vamos para prática, primeiro o que temos que ter em mente é que quando o pacote é entregue para o host remoto acontece o inverso do encapsulamento, segundo quando a comunicação se dar em uma rede Ehternet, o pacote vai se chamar 'Frame Ethernet'. Quando a camada de  transporte  diz que o pacote vai ser enviado se utilizando de UDP ai chamamos  o pacote de  datagrama. Modulo nada mais eh que um software que se comunica  diretamente com o drive de dispositivo (da interface de rede) que age no kernel traduzindo instruções  do periférico para o OS.O modulo  também se  comunica com  aplicativos de rede e outros módulos. Podemos dizer então que o que faz as coisas  acontecerem  no  encapsulamento de dados são os tais módulos. Não se esqueça que os módulos também se comunicam com aplicativos de rede, que nada  mais são que programas que fazem as requisições de conexão, como browsers,  clientes
de FTP, POP, clientes de trojans, etc....:) Lembrando que as  aplicações  não apenas fazem requisições, também ficam esperando as tais...Sacou? =) Os aplicativos de rede
se comunicam com os modulos apenas para lhes entregar as requisições ou para receber as requisições que são trazidas pelos modulos no caso do  recebimento. não entendeu ainda ?  queria deixar isso para parte da programação, porém sei que vai ajudar a visualizar na sua mente como é mais ou menos esse encapsulamento outra coisa, que pena que esse artigo não é direcionado para quem curti a linguagem python, pois só com ela podemos ver esse processo e analizar cada frame, cada pedaço do header, cada fragmento de bytes,isso tudo por causa do interpretador e ruby também podemos fazer isso,  mas deixa isso para um outro paper irei fazer em breve python é só alegria. vamos lá então.Suponhamos que enviamos uma mensagem com seguinte conteúdo, "Vem monstro, Pode vim monstro", na camada de aplicação é claro, logo seguida vem nosso buffer é coloca dentro de um segmento TCP, no segmento TCP pode ser divido em duas partes o header e os dados, lógico depois de codificada pela aplicação veremos isso melhor em breve keep me calm rapaz, fica mais ou menos assim:</p>
<h3>Exemplo 1</h3>
<pre>
                        DATA Dump                         
IP Header
    45 20 00 89 8B 38 40 00 33 06 D4 7D AE 8F 77 5B         E ...8@.3..}..w[
    C0 A8 01 06                                             ....
TCP Header
    1A 0B 95 79 49 BF 5F 41 03 E5 5E 37 50 18 25 B0         ...yI._A..^7P.%.
    B6 87 00 00                                             ....
Data Payload
    3A 6B 61 74 65 60 21 7E 6B 61 74 65 40 75 6E 61         :kate`!~kate@una
    66 66 69 6C 69 61 74 65 64 2F 6B 61 74 65 2F 78         ffiliated/kate/x
    2D 30 30 30 30 30 30 31 20 50 52 49 56 4D 53 47         -0000001 PRIVMSG
    20 23 23 63 20 3A 69 20 6E 65 65 64 20 65 78 61          ##c :i need exa
    63 74 6C 79 20 74 68 65 20 72 69 67 68 74 20 6E         ctly the right n
    75 6D 62 65 72 20 6F 66 20 73 6F 63 6B 73 21 0D         umber of socks!.
    0A                                                      .
</pre>
<h3>Exemplo 2</h3>
<pre>
                        DATA Dump                         
IP Header
    45 20 00 BA DC EC 00 00 30 06 59 73 4A 7D 47 93         E ......0.YsJ}G.
    C0 A8 01 06                                             ....
TCP Header
    00 50 C0 DE BA EC 43 51 94 54 B9 19 50 18 AE DD         .P....CQ.T..P...
    3A E6 00 00                                             :...
Data Payload
    48 54 54 50 2F 31 2E 31 20 33 30 34 20 4E 6F 74         HTTP/1.1 304 Not
    20 4D 6F 64 69 66 69 65 64 0D 0A 58 2D 43 6F 6E          Modified..X-Con
    74 65 6E 74 2D 54 79 70 65 2D 4F 70 74 69 6F 6E         tent-Type-Option
    73 3A 20 6E 6F 73 6E 69 66 66 0D 0A 44 61 74 65         s: nosniff..Date
    3A 20 54 68 75 2C 20 30 31 20 44 65 63 20 32 30         : Thu, 01 Dec 20
    31 31 20 31 33 3A 31 36 3A 34 30 20 47 4D 54 0D         11 13:16:40 GMT.
    0A 53 65 72 76 65 72 3A 20 73 66 66 65 0D 0A 58         .Server: sffe..X
    2D 58 53 53 2D 50 72 6F 74 65 63 74 69 6F 6E 3A         -XSS-Protection:
    20 31 3B 20 6D 6F 64 65 3D 62 6C 6F 63 6B 0D 0A          1; mode=block..
    0D 0A  
</pre>
<p>quase iria esquecendo do Header, nos cabeçalhos um é do IP header que já diz tudo e outro é o TCP header , vai por mim na parte programação explicarei cada um deles, só estou mostrando no intuito de memorizar a leitura associando aos texto as imagens que não deixa de ser um texto kkkk. um professor meu falava sempre no ensino médio. em fim, fica assim os dois:</p>
<h3>IP Header</h3>
<pre>
   |-IP Version        : 4
   |-IP Header Length  : 5 DWORDS or 20 Bytes
   |-Type Of Service   : 32
   |-IP Total Length   : 186  Bytes(tamanho do pacote)
   |-Identification    : 56556
   |-TTL      : 48
   |-Protocol : 6
   |-Checksum : 22899
   |-Source IP        : 192.168.0.100
   |-Destination IP   : 192.168.0.102
</pre>
<h3>TCP Header</h3>
<pre>
   |-Source Port      : 80
   |-Destination Port : 49374
   |-Sequence Number    : 3136045905
   |-Acknowledge Number : 2488580377
   |-Header Length      : 5 DWORDS or 20 BYTES
   |-Urgent Flag          : 0
   |-Acknowledgement Flag : 1
   |-Push Flag            : 1
   |-Reset Flag           : 0
   |-Synchronise Flag     : 0
   |-Finish Flag          : 0
   |-Window         : 44765
   |-Checksum       : 15078
   |-Urgent Pointer : 0
</pre>
<p>o cabeçalho com 20 bytes, isso aplicação vai fazer tudo, claro se você usar o socket normal no modo Noob, já no modo l33t já muda tudo é por nossa conta heheeheh, voltando para entender precisamos saber que Destination IP é que vai analisar a mensagem é muito importante, que é representado em 16 bit vai uma imagem ai pra não chorar:</p>
<img src="/images/posts/cabecalhotcp.jpg">
<p>Com todos os requisitos setando com sucesso, o segmento é encaminhado de Rede, mas não estávamos na camada de aplicação ? é que esqueci de um detalhe o modulo TCP já passa tudo isso, deixando o pacote já pronto para camada de rede, ao chamar na camada de rede o primeiro passo é encapsular o segmento TCP dentro da unidade de dados da camada de rede, e nessa camada onde se verifica tudo aquilo que nós colocamos no header, nesse caso o pacote identifica a origem e o destino, verifica o TTP,flag, protocolo(muito importante), tipo do serviço, versão e etc...  ou seja as informações do header TCP, se liga na imagem:</p>
<img src="/images/posts/cabecalhoipv4.jpg">
<p>O protocolo é muito importante, ele permite ao host destino quando receber o pacote identificar qual tipo de protocolo se tiver um 6 esse protocolo vai ser TCP como vimos no exemplo citado acima,se for 1 quer dizer que o protocolo vai ser ICMP (ping), caso for = 2 será IGMP Protocol e 17 para UDP, o Source IP é o IP que está enviando a requisição, apenas dessa forma com socket l33t podemos colocar o IP que quisermos, ou seja, podemos colocar que a requisição foi enviada pelo google.com.br, isso já abre o leck de possibilidades de attack como DOS, os flood em geral e  etc... e logo em seguida chegamos na camada de enlace, depois de tudo verificado chegamos na camada de enlace e codificamos com a interface de rede wlan0, e for fim ehhehee chega na camada de física que será codificada em onda eletromagnéticas no caso wifi.</p>
<h3>Socket Raw na Prática :D Wellcome a Selva l33t Wilsoooooooon</h3>
<p>Depois de resumir o encapsulamento de dados, iremos para parte prática da coisa... lembrando que python seria, mas interessante pois iriamos ver o pacote ao vivo sendo capturado, mas quem sabe um serie sobre python em breve. let's Go</p>

``` c  
#include <sys/types.h>
#include <stdio.h>
#include <stdlib.h>
#include <sys/socket.h>
#include <errno.h>
#include <unistd.h>
#include <arpa/inet.h> //operações de internet
#include <net/ethernet.h> // ether_header
#include <netinet/in.h> //Internet Protocol family
#include <netinet/ip.h> // declaração ip header
#include <netinet/tcp.h> // declaração para TCP header

// struct do IP HEADER
struct headerTCP {
    u_int32_t src_addr;
    u_int32_t dst_addr;
    u_int8_t padding;
    u_int8_t proto;
    u_int16_t length;
};

struct data_checksum {
    struct headerTCP pshd;
    struct tcphdr tcphdr;
    char payload[1024];
};
unsigned short comp_chksum(unsigned short *addr, int len) {
    long sum = 0;

    while (len > 1) {
        sum += *(addr++);
        len -= 2;
    }

    if (len > 0)
        sum += *addr;

    while (sum >> 16)
        sum = ((sum & 0xffff) + (sum >> 16));

    sum = ~sum;

    return ((u_short) sum);

}
```


<p>primeiro de tudo declaramos os header's, até comentei alguns deles, logo seguida vem duas estruturas para o ip header e tcp header, calma  ficará tudo claro em breve, o checksum é a  soma de verificação, é usado para detectar a corrupção de dados através de uma conexão isso quer dizer que qualquer erro que ocorrer na soma, Se um bit é invertida, um byte mutilado, ou alguma outra maldade acontece com um pacote, ele provavelmente  vai ser quebrado e não conseguirá entrar  os dados para o destino, qual motivo da criação dessa medida, então o checksum é uma garantia que o fluxo de dados está correto. No IPV4 essa soma  verificação para detectar a corrupção de cabeçalhos dos pacotes. ou seja, a origem, o destino e outras meta-dados. isso tudo para proteger o payload.  há e outra coisa o algoritmo de verificação do checksum é o mesmo tanto para TCP e IPV4.  também temos lá em cima a definição de uma estrutura que se você comparar com uma das imagens que citei acima como exemplo vera que nada mais é que a estrutura do nela temos o endereço de origem  e o endereço de destino, u_int32_t src_addr;  u_int32_t dst_addr; , ainda temos o padding né que é opcional nem irei usa-lo, tá mas pra que serve ? então o padding no cabeçalho TCP é usado para garantir que as extremidades do cabeçalho TCP  e dados começa num limite de 32 bits. O Padding ou preenchimento  não vamos generalizar mas quase sempre  consiste em apenas zeros para assegurar a parte do pacote de dados não está perdida.  Proto já diz tudo onde definimos que tipo de protocolo vai ser usado. </p>


``` c
int main(int argc, char *argv[]) {


    // bloco 1
    int sock, bytes, nivel = 1;
    char buffer[1024];
    struct iphdr *ip;
    struct tcphdr *tcp;
    struct sockaddr_in to;
    struct headerTCP p_tcp_header;
    struct data_checksum  checksum_tcp_calc;


    // bloco 2
    if (argc != 2) {
        fprintf(stderr, "Usage: %s ", argv[0]);
        fprintf(stderr, "\n");
        return 1;
    }

    sock = socket(AF_INET, SOCK_RAW, IPPROTO_TCP);
    if (sock == -1) {
        perror("socket() failed");
        return 1;
    }else{
        printf("socket() ok\n");
    }

    if (setsockopt(sock, IPPROTO_IP, IP_HDRINCL, &nivel, sizeof(nivel)) == -1) {
        perror("setsockopt() failed");
        return 2;
    }else{
        printf("setsockopt() ok\n");
    }
     // Apenas para exemplo do comentário abaixo
     //int setsockopt( int s,
    //           int level,
    //            int optname,
    //            const void * optval,
    //            socklen_t optlen );

    //bloco 3
    ip = (struct iphdr*) buffer;
    tcp = (struct tcphdr*) (buffer + sizeof(struct tcphdr));

    int iphdrlen = sizeof(struct iphdr);
    int tcphdrlen = sizeof(struct tcphdr);
    int datalen = 0;
```

Acima vem as definições de algumas coisinhas como, socket que já vimos nos paper passados, e também das estruturas que vamos passar alguns argumentos. dividir em blocos para não perdemos muito tempo na explicação tem muita coisa básico vou focar apenas nas partes que são importante para o paper.
bloco 1 vemos apenas declarações de variais e de estruturas que são fundamentais no pacote.
bloco 2 nessa parte vem muita coisa que já vimos sobre socket, criação e tratamento de erro, um função que até então não trabalhei é a socketopt() vai render umas linhas, o uso dessa função é bastante comum em socket raw lógico, Essa função é usada para setar opções num socket, o primeiro argumento claro, é nosso socket descritor, segundo vem o nível,o nível é usado para manipular opções no nível socket, tá como assim ? Por exemplo, para indicar que uma opção eh para ser interpretado pelo protocolo TCP, nível deve estar setado para o numero do protocolo TCP, que é 6. logo em seguida vem optname, Optname sao as opcoes propriamente ditas que serao setadas para o nosso arquivo socket.Elas sao passadas para o modulo do protocolo apropriado para interpretação. no nosso caso estamos usado o IP_HDRINCL podem nao serem validos caso esteja usando uma kernel antiga. depois o optval (constante nivel) Os parâmetros optval e optlen são usados ​​para acessar os valores de opção para setsockopt (). Para getsockopt () identificam um buffer no qual o valor para a opção solicitada (s) devem ser devolvidos. Para getsockopt (), optlen é um parâmetro de valor para os resultados, inicialmente contendo o tamanho do buffer apontado por optval, e modificado no retorno ao indicam o tamanho real do valor devolvido. Se nenhum valor opção é a ser fornecido ou devolvidos, optval pode ser NULL. no nosso caso, para habilitar uma opção booleana, ou zero se uma opção é para ser desabilitada. na verdade essa função não serve apenas para o que vou explicar e sim para milhões de utilidades, quem sabe em outro paper podemos usa-la e cita-la como um exemplo disso.
bloco 3 apenas associamos as estruturas para preenchimento abaixo você poderá ver melhor como funciona, há também definimos o tamanho das estruturas são bem importante pois vamos precisa-las em breve.

``` c++
   // bloco 1
    ip->frag_off = 0;
    ip->version = 4;
    ip->ihl = 5;
    ip->tot_len = htons(iphdrlen + tcphdrlen);
    ip->id = 0;
    ip->ttl = 40;
    ip->protocol = IPPROTO_TCP;
    ip->saddr = inet_addr("192.168.1.135"); // IP Spoof :D
    ip->daddr = inet_addr(argv[1]); // Destiny IP, lembrei do jogo muito loko Destiny
    ip->check = 0;


    //bloco 2
    tcp->source     = htons(12345);
    tcp->dest       = htons(80);
    tcp->seq        = random();
    tcp->doff       = 5;
    tcp->ack        = 0;
    tcp->psh        = 0;
    tcp->rst        = 0;
    tcp->urg        = 0;
    tcp->syn        = 1;
    tcp->fin        = 0;
    tcp->window     = htons(65535);


    //bloco 3
    p_tcp_header.src_addr = ip->saddr;
    p_tcp_header.dst_addr = ip->daddr;
    p_tcp_header.padding = 0;
    p_tcp_header.proto = ip->protocol;
    p_tcp_header.length = htons(tcphdrlen + datalen);

     checksum_tcp_calc.pshd = p_tcp_header;
     checksum_tcp_calc.tcphdr = *tcp;

    int checksum = comp_chksum((unsigned short*) &checksum_tcp_calc,
            sizeof(struct headerTCP) + tcphdrlen + datalen);

    tcp->check = checksum;

    printf("TCP Checksum: %i\n", checksum);
    printf("Destination : %i\n", ntohs(tcp->dest));
    printf("Source: %i\n", ntohs(tcp->source));
```

Bloco 1 acredito que você já tenha uma noção de tudo isso, temos uma imagem que demonstra isso graficamente estamos falando o Header IP, onde tem algumas informações sobre requisições. indenficações e etc... veremos cada uma deles, primeiro a flag, frag_off O primeiro bit é reservado e definido como 0. DF (não fragmentar) controla a fragmentação do datagrama. MF (Mais fragmentos) Indica se há mais fragmentos de receber. Se este sinalizador definido como zero, significa que este é o último fragmento do datagrama. versão IPV4 né, A versão do protocolo de internet. ip->IHL este campo indica o tamanho do cabeçalho (isso também coincide com o deslocamento para os dados). O valor mínimo para este campo é 5 valor de 5 para o hl significa 20 bytes (5 * 4):D. A parte interessante o resto é o básico de redes requisições há outra coisa colocomos 1 no campo do syn, é claro é nosso tipo de requisição. e finalmente nosso ultimo arguento ip->window. que nada mais é que o campo de deslocamento fragmento, medido em unidades de blocos de oito bytes (64 bits), é de 13 bits de comprimento e especifica o deslocamento de um fragmento em particular em relação ao início do datagrama IP original unfragmented. O primeiro fragmento foi um desvio de zero. Isto permite um deslocamento de (2 máximo 13 - 1) × 8 = 65.528 bytes, o que excede o comprimento máximo de pacotes IP de 65.535 bytes com o tamanho do cabeçalho incluído (65.528 + 20 = 65.548 bytes).
bloco 2 Esse também não tem muita explicação básico de C , apenas atribuição de alguns argumentos do header TCP da uma olhada em umas das figuras que você vai entender tudinho, nesse bloco basicamente preenchemos o HEader TCP e calculamos o Checksum. e depois com o dois Header preenchidos damos um print() no destino e na origem do pacote há lembrando usando socket raw podemos manipular e colocar o IP queremos assim não se sabe quem mandou o pacote, lembra do Nmap, então o argumento source-spoof acho que é assim é isso mesmo meu amigo, funciona quase assim, muitos DOS atttack usa esse tipo de socket para aproveitar do controle que tem usando esse tipo de socket. "a coisa mais linda de Deus um backdoor raw socket " frases do cooler_void >D, da uma olhada nos argumento da função comp_checksum os argumentos são passado contendo o tamanho de cada estruturas.

``` c++
   // bloco 1
   to.sin_addr.s_addr = ip->daddr;
    to.sin_family = AF_INET;
    to.sin_port = tcp->dest;


    // bloco 2
    bytes = sendto(sock, buffer, ntohs(ip->tot_len), 0, (struct sockaddr*) &to,
            sizeof(to));

    if (bytes == -1) {
        perror("sendto() failed");
        return 1;
    }

    recv(sock, buffer, sizeof(buffer), 0);
    printf("TTL= %d\n", ip->ttl);
    printf("Window= %d\n", tcp->window);
    printf("ACK= %d\n", tcp->ack);
    printf("%s:%d\t --> \t%s:%d \tSeq: %d \tAck: %d\n",
                    inet_ntoa(*(struct in_addr*) &ip->saddr), ntohs(tcp->source),
                    inet_ntoa(*(struct in_addr *) &ip->daddr), ntohs(tcp->dest),
                    ntohl(tcp->seq), ntohl(tcp->ack_seq));

    return 0;
}
```

Definir em dois blocos para acabar logo com essa parte :D, também não tem muito o que explicar por aqui, boco 1 básico de socket já falei em outros papers, mas quem não tiver lembrando, atribuirmos o ip destino, a família do socket e porta pra onde o pacote vai ser enviado. Bloco 2a função sendto() já conhecemos, básico de socket, comparamos se o retorno é igual a -1 isso quer dizer que não ouvi resposta do alvo, uma função recv() né é normalmente usado apenas em um ligado socket e é idêntico ao Recvfrom () com um NULL src_addr argumento. passamos o descritor o buffer, né quantidade bytes,e a frag que Especifica o tipo de recepção da mensagem. vem agora a os print() né onde é impresso o TTL que até esqueci de falar dele Time to Live, TTL impede que um pacote de dados de circular indefinidamente e também é um campo de 8 bits veja nas imagens que mostrei logo no começo, O tempo máximo em segundos o datagrama pode permanecer no sistema de internet, cada computador processa o datagrama deve diminuir o valor TTL por pelo menos um. e, se este campo atinge o valor zero, o datagrama deve ser descartado e não mais entregue.
``` c++
// Extra ;D
struct icmpheader {
 unsigned char icmp_type;
 unsigned char icmp_code;
 unsigned short int icmp_cksum;
 unsigned short int icmp_id;
 unsigned short int icmp_seq;
}; / *  icmp: 8 bytes (= 64 bits) * /
```
O que temos acima é a estrutura no caso do ICMP, o implemento do ICMP muda um pouco, mas segue a mesma linhas. icmp_type o tipo de mensagem, por exemplo 0 - echo resposta, 8 - echo pedido, 3 - destino inacessível. olhar em para todos os tipos.
icmp_code isso é significativo ao enviar uma mensagem de erro (unreach), e especifica o tipo de erro. novamente, consulte o arquivo de inclusão para mais.
icmp_cksum a soma de verificação para o cabeçalho icmp + dados. mesmo que a soma de verificação IP. Nota: Os 32 bits seguintes em um pacote ICMP pode ser usado de muitas maneiras diferentes. icmp_id usado na solicitação de eco / resposta mensagens, para identificar o pedido. icmp_seq: identifica a seqüência de mensagens de eco, se mais de um é enviado.

O User Datagram Protocol é um protocolo de transporte para as sessões que precisa dados do Exchange. Ambos os protocolos de transporte, UDP e TCP fornecer 65535 diferente portas de origem e destino. A porta de destino é usado para conectar-se um serviço específico nessa porta. Ao contrário do TCP, UDP não é confiável, uma vez que ele não usa números de seqüência e conexões stateful. Isto significa UDP datagramas podem ser falsificados, e pode não ser confiável (por exemplo, eles podem ser perdidos despercebido), uma vez que não são reconhecidos usando respostas e números de seqüência. vamos dar uma olhada no header UDP.

``` c++
struct udpheader {
 unsigned short int uh_sport;
 unsigned short int uh_dport;
 unsigned short int uh_len;
 unsigned short int uh_check;
}; / * Comprimento total cabeçalho udp: 8 bytes (= 64 bits) * /
```
uh_sport: A porta de origem que uma ligação do cliente () s para, eo servidor contactado responderemos de volta para a fim de direcionar suas respostas para o cliente.
uh_dport: A porta de destino que um servidor específico pode ser contactado por diante.
uh_len: O comprimento de dados do cabeçalho e carga UDP em bytes.
uh_check: A soma de verificação de cabeçalho e dados, consulte soma de verificação IP.

``` c++
Simples Pacote Sniffer

#include <stdio.h>
#include <stdlib.h>
#include <errno.h>
#include <unistd.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <linux/if_ether.h>
#include <net/if.h>
#include <linux/filter.h>
#include <sys/ioctl.h>
#include <string.h>

// by: pokerbirch
int main(int argc, char **argv) {
    int sock, i;
    unsigned char buffer[2048];
    unsigned char *iphead, *ethhead;
    struct ifreq ethreq;

    // NOTE: use TCPDUMP to build the filter array.
    // set filter to sniff only port 443
    // $ sudo tcpdump -dd port 443

    struct sock_filter BPF_code[] = {
        { 0x28, 0, 0, 0x0000000c },
        { 0x15, 0, 8, 0x000086dd },
        { 0x30, 0, 0, 0x00000014 },
        { 0x15, 2, 0, 0x00000084 },
        { 0x15, 1, 0, 0x00000006 },
        { 0x15, 0, 17, 0x00000011 },
        { 0x28, 0, 0, 0x00000036 },
        { 0x15, 14, 0, 0x000001bb },
        { 0x28, 0, 0, 0x00000038 },
        { 0x15, 12, 13, 0x000001bb },
        { 0x15, 0, 12, 0x00000800 },
        { 0x30, 0, 0, 0x00000017 },
        { 0x15, 2, 0, 0x00000084 },
        { 0x15, 1, 0, 0x00000006 },
        { 0x15, 0, 8, 0x00000011 },
        { 0x28, 0, 0, 0x00000014 },
        { 0x45, 6, 0, 0x00001fff },
        { 0xb1, 0, 0, 0x0000000e },
        { 0x48, 0, 0, 0x0000000e },
        { 0x15, 2, 0, 0x000001bb },
        { 0x48, 0, 0, 0x00000010 },
        { 0x15, 0, 1, 0x000001bb },
        { 0x6, 0, 0, 0x00000060 },
        { 0x6, 0, 0, 0x00000000 }
    };
    struct sock_fprog Filter;

    Filter.len = sizeof(BPF_code) / 8;
    Filter.filter = BPF_code;

    if ((sock = socket(PF_PACKET, SOCK_RAW, htons(ETH_P_IP))) == -1) {
        perror("socket");
        exit(1);
    }

    // set network card to promiscuos
    strncpy(ethreq.ifr_name, "eth0", IFNAMSIZ);
    if (ioctl(sock,SIOCGIFFLAGS, ðreq) == -1) {
        perror("ioctl");
        close(sock);
        exit(1);
    }
    ethreq.ifr_flags |= IFF_PROMISC;
    if (ioctl(sock, SIOCSIFFLAGS, ðreq) == -1) {
        perror("ioctl");
        close(sock);
        exit(1);
    }

    // attach filter to socket
    if(setsockopt(sock, SOL_SOCKET, SO_ATTACH_FILTER, &Filter, sizeof(Filter)) == -1) {
        perror("setsockopt");
        close(sock);
        exit(1);
    }

    while (1) {
        printf("----------------------\n");
        i = recvfrom(sock, buffer, sizeof(buffer), 0, NULL, NULL);
        printf("%d bytes read\n", i);

        // check header size: Ethernet = 14, IP = 20, TCP = 8 (sum = 42)
        if (i < 42) {
            perror("recvfrom():");
            printf("Incomplete packet (errno is %d)\n", errno);
            close(sock);
            exit(0);
        }
        ethhead = buffer;
        iphead = buffer + 14; // (skip ethernet  header)

        if (*iphead == 0x45) {
            printf("Source Address: %d.%d.%d.%d, Port: %d\n",
                iphead[12], iphead[13], iphead[14], iphead[15], (iphead[20] << 8) + iphead[21]);
            printf("Dest Address: %d.%d.%d.%d, Port: %d\n",
                iphead[16], iphead[17], iphead[18], iphead[19], (iphead[22] << 8) + iphead[23]);
        }
    }
    // clean up
    close(sock);
}
```
Esse programa apenas captura dados Ethernet , a estrutura BPF_code faz justamente isso configura qual tipo de pacote com as especificações de cada pacote, a função setsockopt é justamente para isso mesmo, setar um filtro no socket. iphead == 0x45 isso significa que é apenas para imprimir pacotes IPV4, O "5" em "45" é o tamanho do cabeçalho IP, em unidades de 4 bytes, de modo que é de 20 bytes. se você deseja filtrar os dados que estão sendo transportados através de TCP, você tem que pular o cabeçalho TCP (que é de comprimento variável, assim como é o cabeçalho IP). o resto é print() mostrando de onde e pra onde está indo o pacote.

Conclusão

O objetivo desse artigo foi tentar de uma forma interativa como tudo funciona na criação e encapsulamento de dados. além disso, aliando a programação fica mais interessante ainda, pois podemos desenvolver outros pacotes com tipos de protocolos diferentes, lembrando que isso não é tudo tem muita coisa que deixei passar para não perder o foco, detalhei apenas as coisas mais importante para um conhecimento prévio sobre socket raw, também aprendendo a inserir qualquer datagrama baseada em protocolo IP para o tráfego de rede. Isto é útil, por exemplo, para construir scanners Tomada matérias como nmap, falsificar ou para realizar operações que precisam enviar soquetes simples. Basicamente, você pode enviar qualquer pacote a qualquer momento, enquanto utilizando as funções de interface para seus sistemas IP- pilha (ligar, escrever, bind, etc.) que você não tem controle direto sobre os pacotes. O conceito básico de socket raw de baixo nível é para enviar um único pacote de uma só vez, com todos os cabeçalhos de protocolo preenchidos pelo programa (ao invés de o kernel).Unix fornece dois tipos de tomadas que permitem o acesso directo à rede. há algumas funções que detalhei é exatamente porque vamos usa-las em outros artigos que irei escrever, assim não preciso perder muito tempo explicando cada uma delas. hope i helped you , see you next paper...bye artigo escrito ao som de Thriftworks :D

<pre>
Referências:
http://pt.wikipedia.org/wiki/TCP/IP
http://www.opennet.ru/base/sec/p49-15.txt.html
http://ubuntuforums.org/showthread.php?t=954426
http://nunix.fr/index.php/programmation/1-c/58-rawsocket-to-forge-udp-packets
http://jkolb.com.br/wp-content/uploads/2016/06/modelo-osi2.png
</pre>
