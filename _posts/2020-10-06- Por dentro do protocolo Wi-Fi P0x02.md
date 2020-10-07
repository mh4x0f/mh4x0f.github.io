---
layout: post
title: Por dentro do protocolo Wi-Fi P0x02 
categories: [python3]
tags: [hacking mh4x0f security tools python3 scapy wireless protocolo dot11 802.11 IEE aircrack-ng packets airodump-ng Reassociation request Dot11ProbeReq ]
fullview: true
comments: true
---


No artigo anterior aprendemos como capturar informações importante de uma rede wireless (Wi-fi) tais como: (SSID, BSSID, AUTH e o CHANNEL ), essa informações são as mais simples e não tem muito impacto ou alguma **info** que podemos usar como vantagem em algum tipo de ataque, a não ser que a rede seja do tipo **WEP** bem difícil hoje em dia. Porém já sabemos como achar um cliente daquela rede, isso é uma informação importante. let's go... 

### Introdução

![airodump_ng](/images/P0x01/airdump-ng.png)

A parte de baixo da imagem acima foi o que fiquei devendo pra você, vou mostrar como funciona e como podemos capturar. Estamos diante de um tipo de pacote muito específico que só e metido quando você não está conectado a nenhuma rede e que tem informações importantes, Esses pacotes são chamados de **Probe Requests**, é um tipo de pacote que emitimos quando estamos procurando por redes já conhecidas na rede. Então se eu capturar esses pacotes, tenho acesso as redes pessoais de um determinado alvo ?

![gratidao](/images/P0x02/gratidao_zoipucu.jpg)

Senhoras e senhores : Wellcome the jungle

### Porque o PR (Probe Request) é um problema ?

Sempre que o Wi-Fi do seu dispositivo está ligado, mas não conectado a uma rede, ele transmite abertamente os SSIDs (nomes de rede) de todas as redes associadas anteriormente em uma tentativa de se conectar a uma delas. Esses pequenos pacotes, chamados de solicitações de sondagem (Probe Request- PR), podem ser visualizados publicamente por qualquer pessoa na área executando um software de detecção trivialmente simples, e você ficaria surpreso com o quão única é sua lista de redes.
fonte: https://medium.com/@brannondorsey/wi-fi-is-broken-3f6054210fa5

**Brannon Dorsey** diz ser simples e trivial, e realmente é, mas ele esqueceu que ele ta usando o **tcmpdump** para filtrar os pacotes do tipo PR, fala sério né, faz isso em C negão e como a gente diz na bahia "VAI VER BICHO!".

Mas no geral, o cara ta certo (python é menos de 10 linhas). Isso pode ser um perigo, pense ai, tem saber o lugar que você frequenta pode ser um início de um ataque direcionado.

### Por onde começar ?

 Vamos lá, a primeira coisa que precisamos saber é que os pacotes são de um tipo específico, então só precisamos captura-los, semelhante o que fizemos no capitulo anterior, porem agora vai ser bem mais simples, o que temos que entender é como funciona o frame do PR, ou melhor, entender qual o parser exato do MAC header.

 Eu não mencionei antes mas existe subtypes que define o tipo do frame que está sendo transmitido, veja so algums deles:
``` sh
 | Frames                                          | Subtype Bits |
|-------------------------------------------------|--------------|
| Association request                             | 0000         |
| Association response                            | 0001         |
| Reassociation request                           | 0010         |
| Reassociaiton response                          | 0011         |
| Probe request                                   | 0100         |
| Probe response                                  | 0101         |
| Beacon                                          | 1000         |
| ATIM (Announcement traffic indication message ) | 1001         |
| Disassociation                                  | 1010         |
| Authentication                                  | 1011         |
| Deauthentication                                | 1100         |
| Action                                          | 1101         |
```

O importante para nós é o PR que o subtipo é o 4 em decimal, estão temos que começar por ai. mas o scapy na mais nova versão trouxe um recurso melhor para fazer isso, que é o **Dot11ProbeReq**. veja só:

``` py
class Dot11ProbeReq(_Dot11EltUtils):
    name = "802.11 Probe Request"
```

Então essa será a forma que iremos filtrar apenas os pacotes que são definidos por essa class, da mesma forma que podemos capturar esses pacotes também podemos transmiti-los fácilmente só seguir a montagem de pacotes e inserindo as informações do Dot11Elt que já sabemos como funciona. Vamos lá 

``` py
    if (
        pkt.haslayer(Dot11ProbeReq)
        and "\x00".encode() not in pkt[Dot11ProbeReq].info
    ):
        essid = pkt[Dot11ProbeReq].info
    else:
        essid = "Hidden SSID"
    client = pkt[Dot11].addr2

    if client not in self.clients:
        self.clients[client] = []
```

Então, como diria um velho amigo "morreu bucha de sena". Se você quer onde o target costuma acessar basta deixar capturando. Mas como sempre digo aqui a gente entende os porquês da coisa. Vamos lá, estamos pegando o **addr2** e associando ao cliente que está mandando esse pacote. Na verdade o **addr2** nesse caso é o "Transmitter Address" ou melhor dizendo **source address**, que é realmente o Mac Address de quem está mandando o pedido, essa informação está contida no **Frame Control** como já vimos no artigo anterior.

- 1 Mas porque esses pacotes são enviados ? 
- 2 Quem responde esses pedidos ? 

Essas são algumas da perguntas que você pode descobrir fazendo uma análise com **wireshark** e interpretrandos as trocas de pacotes. Mas vamos lá, a primeira é simples é um pedido, ou seja, o cliente ou estação está procurando pelo SSID, assim ele envia um frame chamado probe request. A segunda é complementar a primeira, o AP que tiver o SSID envia um Probe Response. Lembrando fique sempre com o modo **Red Team** ligado, essas resposta também abre portas para a criação de outros tipos de ataques (**Boruto**) que iremos abortar aqui, mas sem spoliers, keep doing hack the planet. Ainda tem uma terceira coisa a completar aqui, primeiro que podemos saber mais ou menos qual tipo de aparelho está mandandos esse pacote, primeiro que já sabemos o MAC Address do aparelho e com o **mac** também podemos saber com os 3 primeiros 3 bytes do endereço, que geralmente são definidos pelos fabricantes para deteminar uma serie de aparelhos, são mais conhecidos como **Mac Vendors** existe vários sites que fazem esse trabalho, ainda existe módulo do python focado em resolver esses endereço. Dessa forma, é possível saber por exemplo se é Sansung,Motorola, Xaomi e etc.

### Como saber quem está contectado ao AP

Essa pergunta parece obvia, porém pode ser confusa também. Porque pense comigo, como eu vou saber quem ta mandando pacotes ou recebendo pacotes ? como eu vou conseguir filtrar quem está mandando e associar a quem recebe esses dados ?. Se você respondeu todas essas perguntas essa é a hora de sair daqui porque esse artigo não serve pra você. Agora se surgiu mais perguntas ainda, certamente você vai conseguir responde-las ao longo desse artigo. 

Primeiro que, deve existe um tipo de frame específico pra isso, mas não para saber quem está conectado ao AP, e sim um **pedido**, porque você não fica conectado o tempo todo, veja a tebela dos frame na sessão anterior. Olhando bem a estrutura do **airodump-ng** tem uma columa chamada **FRAME**, então esse é o caminho, vamos descobrir qual tipo de frame é esse que estamos tentando capturar para saber quem está associado ao AP. Bom, esse frame é chamado de **Reassociation request/response** ou RR.

### Reassociation request 

Reassociation request é enviado apenas por um STA (cliente) para um AP e usado quando o STA já está associado. Quando uma estação se move da área de cobertura de um ponto de acesso para outro, ela usa o processo de reassociação para informar a rede 802.11 de sua nova localização. Assim que o AP receber a solicitação de reassociação, ele reconhecerá enviando um ACK.

![RR](/images/P0x02/reassoation-request.png)

O que acontece é, para confirmar se realmente o STA está conectado ao AP é saber apenas o pacote reass. request, porque a confirmação (Reass. Response) não é necessário pois esse tipo de pacote só é emitido quando o cliente está conectado ao AP, porém existe a possibilidade de alguém transmitir esse pacote da mesma forma que você está capturando os dados, ou seja, esse pacote pode ser forjado e emitido por alguém próximo.

```
Então é so capturar os pacotes e "meter dança" como a gente diz aqui na bahia ?
```

Existe um protocolo chamado EAP (Extensible Authentication Protocol), também conhecido como EAPOL **RFC 3748**. Esse protocolo é utilizado pelo RADIUS, que também tem os frame reass. requst/ response, então precisamos ignorar esses tipos de pacotes, veja so: 

``` python
DOT11_REQUEST_SUBTYPE = 2 

if ( pkt.haslayer(Dot11) and pkt.getlayer(Dot11).type == DOT11_REQUEST_SUBTYPE
    and not pkt.haslayer(EAPOL)):
    sender = pkt.getlayer(Dot11).addr2
    receiver = pkt.getlayer(Dot11).addr1
```

Como já sabemos o **addr2** é o endereço de origem e o **addr1** é o endereço destino, ou melhor, os endereços do remetente e do destinatário. veja que ainda não temos a informação do AP, ou seja, o BSSID dos APs na região. Por isso esse trecho de código, precisa ser executado e alinhado com a captura dos BSSID. Então agora precisamos verificar se o sender ou receiver é um AP, se for um AP, logo, concluimos que é um cliente STA mandando um pedido. vamos lá,

``` python
        if ( pkt.haslayer(Dot11) and pkt.getlayer(Dot11).type == DOT11_REQUEST_SUBTYPE
            and not pkt.haslayer(EAPOL)):

            sender = pkt.getlayer(Dot11).addr2
            receiver = pkt.getlayer(Dot11).addr1
            if sender in self.aps.keys():
                if not receiver in self.whitelist:
                    self.aps[sender]["STA"] = {
                        "Frames": 1,
                        "BSSID": sender,
                        "Station": receiver,
                        "PWR": self.getRSSIPacketClients(pkt),
                    }
                if "STA" in self.aps[sender]:
                    self.aps[sender]["STA"]["Frames"] += 1
                    self.aps[sender]["STA"]["PWR"] = self.getRSSIPacketClients(pkt)
            elif receiver in self.aps.keys():
                if not sender in self.whitelist:
                    self.aps[receiver]["STA"] = {
                        "Frames": 1,
                        "BSSID": receiver,
                        "Station": sender,
                        "PWR": self.getRSSIPacketClients(pkt),
                    }
                if "STA" in self.aps[receiver]:
                    self.aps[receiver]["STA"]["Frames"] += 1
                    self.aps[receiver]["STA"]["PWR"] = self.getRSSIPacketClients(
                        pkt
                    )
```

Você deve está se perguntando "como que posso verificar se tudo é verdade ?". Simples, existem varias formas de verificar isso, uma delas é verificar no código do **airodump-ng** (recomendo fortemente) que certamente não é nada parecido com essa implementação, muito mais completa e complexa. Uma forma simple de verificar isso é o MAC vendor do receiver ou sender que você está capturando, existem vários módulos do python capaz de fazer isso, uma delas é o [mac_vendor_lookup](https://pypi.org/project/mac-vendor-lookup/), você pode instalar com um simples.

``` sh
$ pip install mac-vendor-lookup
```

Basic Usage:

``` python
from mac_vendor_lookup import MacLookup

print(MacLookup().lookup("00:80:41:12:FE"))
```

Agora só adaptar ao código acima:

``` python
        if ( pkt.haslayer(Dot11) and pkt.getlayer(Dot11).type == DOT11_REQUEST_SUBTYPE
            and not pkt.haslayer(EAPOL)):

            sender = pkt.getlayer(Dot11).addr2
            receiver = pkt.getlayer(Dot11).addr1
            if sender in self.aps.keys():
                if not receiver in self.whitelist:
                    self.aps[sender]["STA"] = {
                        "Frames": 1,
                        "BSSID": sender,
                        "Vendor": MacLookup().lookup(receiver),
                        "Station": receiver,
                        "PWR": self.getRSSIPacketClients(pkt),
                    }
                if "STA" in self.aps[sender]:
                    self.aps[sender]["STA"]["Frames"] += 1
                    self.aps[sender]["STA"]["PWR"] = self.getRSSIPacketClients(pkt)
            elif receiver in self.aps.keys():
                if not sender in self.whitelist:
                    self.aps[receiver]["STA"] = {
                        "Frames": 1,
                        "BSSID": receiver,
                        "Station": sender,
                        "Vendor": MacLookup().lookup(sender),
                        "PWR": self.getRSSIPacketClients(pkt),
                    }
                if "STA" in self.aps[receiver]:
                    self.aps[receiver]["STA"]["Frames"] += 1
                    self.aps[receiver]["STA"]["PWR"] = self.getRSSIPacketClients(
                        pkt
                    )
```

Ok, Esse recurso não foi adicionado ao **airodump-ng**, não sei o porquê, me parece ser um recurso útil que irei, implementa-lo no módulo de scanner do **wifipumpkin3**. O método **getRSSIPacketClients** é um método que apenas volta um valor o RSSI emitido pelo cliente, essa informação pode ser útil também, até porque com ela podemos saber qão distante o cliente está do AP.


### Resultado

Certo, temos um exemplo aqui:

![wifipumpkin3](/images/P0x02/wifideauth2.jpg)

Esse exemplo é fruto de uma [demotração de uso do wifipumpkin3](https://reigadaopsec.com/how-to-perform-a-wifi-deauthentication-attack-with-wifipumpkin3/) feita por Roberto, ferramenta sigo mantendo e faz parte do meu time P0cL4bs Team, onde tem toda essa implementação funcional para diferentes tipos de ataques. código completo:

``` python
from wifipumpkin3.core.common.terminal import ModuleUI
from wifipumpkin3.core.config.globalimport import *
from wifipumpkin3.core.utility.printer import (
    display_messages,
    setcolor,
    display_tabulate,
)
from random import randrange
import time, signal, sys
from multiprocessing import Process
from scapy.all import *
from wifipumpkin3.core.common.platforms import Linux
from tabulate import tabulate

# This file is part of the wifipumpkin3 Open Source Project.
# wifipumpkin3 is licensed under the Apache 2.0.

# Copyright 2020 P0cL4bs Team - Marcos Bomfim (mh4x0f)

# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at

# http://www.apache.org/licenses/LICENSE-2.0

# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

PROBE_REQUEST_TYPE = 0
PROBE_REQUEST_SUBTYPE = 4
DOT11_REQUEST_SUBTYPE = 2


class ModPump(ModuleUI):
    """ Scan WiFi networks and detect devices"""

    name = "wifiscan"

    options = {
        "interface": ["wlanx", "Name network interface wireless "],
        "timeout": [0, "Time duration of scan network wireless (ex: 0 infinty)"],
    }
    completions = list(options.keys())

    def __init__(self, parse_args=None, root=None):
        self.parse_args = parse_args
        self.root = root
        self.name_module = self.name
        self.whitelist = ["00:00:00:00:00:00", "ff:ff:ff:ff:ff:ff"]
        self.aps = {}
        self.clients = {}
        self.table_headers_wifi = [
            "CH",
            "SSID",
            "BSSID",
            "RSSI",
            "Privacy",
        ]
        self.table_headers_STA = ["BSSID", "STATION", "PWR", "Frames", "Probe"]
        self.table_output = []
        super(ModPump, self).__init__(parse_args=self.parse_args, root=self.root)

    def do_run(self, args):
        """ execute module """
        print(
            display_messages(
                "setting interface: {} monitor momde".format(
                    setcolor(self.options.get("interface")[0], color="green")
                ),
                info=True,
            )
        )
        self.set_monitor_mode("monitor")
        print(display_messages("starting Channel Hopping ", info=True))
        self.p = Process(
            target=self.channel_hopper, args=(self.options.get("interface")[0],)
        )
        self.p.daemon = True
        self.p.start()
        print(display_messages("sniffing... ", info=True))
        sniff(
            iface=self.options.get("interface")[0],
            prn=self.sniffAp,
            timeout=None
            if int(self.options.get("timeout")[0]) == 0
            else int(self.options.get("timeout")[0]),
        )
        self.p.terminate()
        self.set_monitor_mode()
        print(display_messages("thread sniffing successfully stopped", info=True))

    def channel_hopper(self, interface):
        while True:
            try:
                channel = randrange(1, 11)
                os.system("iw dev %s set channel %d" % (interface, channel))
                time.sleep(1)
            except KeyboardInterrupt:
                break

    def handle_probe(self, pkt):
        if (
            pkt.haslayer(Dot11ProbeReq)
            and "\x00".encode() not in pkt[Dot11ProbeReq].info
        ):
            essid = pkt[Dot11ProbeReq].info
        else:
            essid = "Hidden SSID"
        client = pkt[Dot11].addr2

        if client in self.whitelist or essid in self.whitelist:
            return

        if client not in self.clients:
            self.clients[client] = []

        if essid not in self.clients[client]:
            self.clients[client].append(essid)
            self.aps["(not associated)"] = {}
            self.aps["(not associated)"]["STA"] = {
                "Frames": 1,
                "BSSID": "(not associated)",
                "Station": client,
                "Probe": essid,
                "PWR": self.getRSSIPacketClients(pkt),
            }

    def getRSSIPacket(self, pkt):
        rssi = -100
        if pkt.haslayer(Dot11):
            if pkt.type == 0 and pkt.subtype == 8:
                if pkt.haslayer(Dot11Beacon) or pkt.haslayer(Dot11ProbeResp):
                    rssi = pkt[RadioTap].dBm_AntSignal
        return rssi

    def getRSSIPacketClients(self, pkt):
        rssi = -100
        if pkt.haslayer(RadioTap):
            rssi = pkt[RadioTap].dBm_AntSignal
        return rssi

    def getStationTrackFrame(self, pkt):
        if (
            pkt.haslayer(Dot11)
            and pkt.getlayer(Dot11).type == DOT11_REQUEST_SUBTYPE
            and not pkt.haslayer(EAPOL)
        ):

            sender = pkt.getlayer(Dot11).addr2
            receiver = pkt.getlayer(Dot11).addr1
            if sender in self.aps.keys():
                if Linux.check_is_mac(receiver):
                    if not receiver in self.whitelist:
                        self.aps[sender]["STA"] = {
                            "Frames": 1,
                            "BSSID": sender,
                            "Station": receiver,
                            "Probe": "",
                            "PWR": self.getRSSIPacketClients(pkt),
                        }
                    if "STA" in self.aps[sender]:
                        self.aps[sender]["STA"]["Frames"] += 1
                        self.aps[sender]["STA"]["PWR"] = self.getRSSIPacketClients(pkt)

            elif receiver in self.aps.keys():
                if Linux.check_is_mac(sender):
                    if not sender in self.whitelist:
                        self.aps[receiver]["STA"] = {
                            "Frames": 1,
                            "BSSID": receiver,
                            "Station": sender,
                            "Probe": "",
                            "PWR": self.getRSSIPacketClients(pkt),
                        }
                    if "STA" in self.aps[receiver]:
                        self.aps[receiver]["STA"]["Frames"] += 1
                        self.aps[receiver]["STA"]["PWR"] = self.getRSSIPacketClients(
                            pkt
                        )

    def handle_beacon(self, pkt):
        if not pkt.haslayer(Dot11Elt):
            return

        essid = (
            pkt[Dot11Elt].info
            if "\x00".encode() not in pkt[Dot11Elt].info and pkt[Dot11Elt].info != ""
            else "Hidden SSID"
        )
        bssid = pkt[Dot11].addr3
        client = pkt[Dot11].addr2
        if (
            client in self.whitelist
            or essid in self.whitelist
            or bssid in self.whitelist
        ):
            return

        try:
            channel = int(ord(pkt[Dot11Elt:3].info))
        except:
            channel = 0

        rssi = self.getRSSIPacket(pkt)

        p = pkt[Dot11Elt]
        capability = p.sprintf(
            "{Dot11Beacon:%Dot11Beacon.cap%}\
                {Dot11ProbeResp:%Dot11ProbeResp.cap%}"
        )

        crypto = set()
        while isinstance(p, Dot11Elt):
            if p.ID == 48:
                crypto.add("WPA2")
            elif p.ID == 221 and p.info.startswith("\x00P\xf2\x01\x01\x00".encode()):
                crypto.add("WPA")
            p = p.payload

        if not crypto:
            if "privacy" in capability:
                crypto.add("WEP")
            else:
                crypto.add("OPN")

        enc = "/".join(crypto)
        self.aps[bssid] = {
            "ssid": essid,
            "channel": channel,
            "capability": capability,
            "enc": enc,
            "rssi": rssi,
        }

    def showDataOutputScan(self):
        os.system("clear")
        self.table_output = []
        self.table_station = []
        for bssid, info in self.aps.items():
            if not "(not associated)" in bssid:
                self.table_output.append(
                    [info["channel"], info["ssid"], bssid, info["rssi"], info["enc"]]
                )
        display_tabulate(self.table_headers_wifi, self.table_output)
        print("\n")
        for bssid, info in self.aps.items():
            if "STA" in info:
                self.table_station.append(
                    [
                        info["STA"]["BSSID"],
                        info["STA"]["Station"],
                        info["STA"]["PWR"],
                        info["STA"]["Frames"],
                        info["STA"]["Probe"],
                    ]
                )
        if len(self.table_station) > 0:
            display_tabulate(self.table_headers_STA, self.table_station)

        print(display_messages("press CTRL+C to stop scanning", info=True))

    def sniffAp(self, pkt):
        self.getStationTrackFrame(pkt)
        if (
            pkt.haslayer(Dot11Beacon)
            or pkt.haslayer(Dot11ProbeResp)
            or pkt.haslayer(Dot11ProbeReq)
        ):

            if pkt.type == PROBE_REQUEST_TYPE and pkt.subtype == PROBE_REQUEST_SUBTYPE:
                self.handle_probe(pkt)

            if pkt.haslayer(Dot11Beacon) or pkt.haslayer(Dot11ProbeResp):
                self.handle_beacon(pkt)

            self.showDataOutputScan()

    def set_monitor_mode(self, mode="manager"):
        if not self.options.get("interface")[0] in Linux.get_interfaces().get("all"):
            print(display_messages("the interface not found!", error=True))
            sys.exit(1)
        os.system("ifconfig {} down".format(self.options.get("interface")[0]))
        os.system("iwconfig {} mode {}".format(self.options.get("interface")[0], mode))
        os.system("ifconfig {} up".format(self.options.get("interface")[0]))
```

### Conclusão 

Portanto, chegamos a conclusão que é tão complicado como parece, conseguimos replicar boa parte do **airodump-ng** além de implementar um recurso novo que não está disponível na versão oficial da ferramenta. Claro, a ferramenta oficial da suite do aircrack-ng implementa muito mais que isso, além de enviar e filtar as informação com mais qualidade e precisão já que estamos falando de C, ideal para trabalhar com network e pequenas estuturas de dados.

Espero ter ajudado alguém com essa pequena análise de uma grande ferramenta para segurança ofensiva e acredito que se não fosse ela anos atrás, eu não teria internet para começar nesse mundo (isso tem história). 

Qual duvida ou crítica pode largar ai nos comentários.

keep doing, hack the planet. 
Até um futuro próximo!


Livros que recomendo:
- Learn Python Network Programming 

Curso: https://www.udemy.com/course/construindo-security-tools-em-python/

Referências:

```
https://www.willhackforsushi.com/papers/80211_Pocket_Reference_Guide.pdf 
https://www.semfionetworks.com/uploads/2/9/8/3/29831147/wireshark_802.11_filters_-_reference_sheet.pdf
https://mrncciew.com/2014/10/28/cwap-reassociation-reqresponse/
https://pypi.org/project/mac-vendor-lookup/
https://www.vocal.com/secure-communication/eapol-extensible-authentication-protocol-over-lan/
https://github.com/P0cL4bs/wifipumpkin3/blob/dev/wifipumpkin3/modules/wifi/wifiscan.py
```