---
layout: post
title: Por dentro do protocolo Wi-Fi P0x01 
categories: [python3]
tags: [hacking mh4x0f security tools python3 scapy wireless protocolo dot11Elt dot11 802.11 IEE aircrack-ng packets ]
fullview: true
comments: true
---


## Motivação 

Já mencionei sobre o tal do scapy (sim, outro artigo sobre python ) e a facilidade de construir pacotes de rede para diferentes tipos de protocolos, inclusive no artigo anterior podemos sentir o poder que esse módulo pode proporcionar. Dessa vez, vamos mais a fundo e entender como funciona a troca de pacotes em uma rede wireless, e claro com o modo Red Team ligado.
O que me motivou a escrever sobre esse tema foi a falta de informações sobre isso na net tanto em português como em english, resolvi documentar algumas coisas importantes.

## Introdução

Nessa serie vamos começar já com os motores ligados, sem introdução ao protocolo e coisas de base, já que as poucas pessoas que me acomponham, já tem uma boa base (porque ninguem tem dúvidas). Vamos falar de IEEE 802.11, talvez um pouco mais profundo no futuro. Você já ouviu falar em aircrack-ng provalmenete, mas você realmente sabe como ele captura redes disponíveis em um determido local ?. Para responder essa pergunta a gente precisa de uma das coisas abaixo: 

- Saber bem C, network e ter tempo pra caralho 
- Saber python, network e  instalar o scapy.

Se você quer saber como funciona as coisas por baixo dos panos, aqui é seu lugar meu amigo. let's go...

## O que vamos fazer aqui ?

Essa é uma boa pergunta, e já respondendo, vamos capturar informações de redes wireless disponíveis em um determinado local,ou melhor, vamos replicar parte da ferramenta aircrack-ng usando o módulo scapy do python.

## IEEE 802.11

A rede sem fio IEEE 802.11, que também é conhecida como rede Wi-Fi, foi uma das grandes novidades tecnológicas dos últimos anos. Atuando na camada física, o 802.11 define uma série de padrões de transmissão e codificação para comunicações sem fio, sendo os mais comuns: FHSS (Frequency Hopping Spread Spectrun), DSSS (Direct Sequence Spread Spectrum) e OFDM (Orthogonal Frequency Division Multiplexing). Atualmente, é o padrão de fato em conectividade sem fio para redes locais (wiki). 


##  O scanner (Aircrack-ng)

O aircrack-ng é feito em C puro (lindo), então claramente a performace vai ser muito melhor, mas a ideia aqui é aprender um pouco mais como funciona o processo de scanner. 

!["airdodump"](/images/P0x01/airdump-ng.png)

O que temos aqui é uma sessão do airdoump-ng, ferramenta que faz parte do aircrack-ng focada em realizar o mapeamento das redes wireless, a primeira parte são os dados das redes disponíveis, esses dados são : (SSID, BSSID, Channel, tipo de encriptação, tipo autenfificação). A segunda parte são os pacotes do tipo probe request, que são os pacotes emitidos pelos clientes como por exemplo um smartphone, pc ou anywhere. Mas vamos deixar os PR (probe request) de lado nessa parte inicial, apesar deles serem muito importante pra nois no futuro.

## Ambiente 

O precisamos aqui é ter **root**, pois precisamos manipular pacotes e tambem mudar o modo de operação da nossa placa wifi, seja ela externa ou não, o acesso **root** sera essencial aqui. O python 3 qualquer versão que o **scapy** funcionar (acima da 2.3), e não menos importante o conhecimento básico de redes de computadores.


### requirements

- Linux [qualquer um negão(a)!]  #se voce usa runwidows sai daqui! 
- Python 3.7+
- Scapy 2.4.3+
- Placa Wireless USB opcional

A placa wireless você pode usar a do seu notebook ou computador desde que ela tenha suporte a pacotes injection ou melhor dizendo ao modo monitor. let's go..

## Packets on the fly - Python 

Na imagem acima vimos que tem uma coluna chamada **beacons**, vou contar um segredo pra você,são eles que eremos capturar, pois eles tem as informações de cada AP (Access Point) que precisamos. então vamos comecar filtrando eles:

``` python 
    if not pkt.haslayer(Dot11Elt):
        return
```

Mas o que seria esse tal do **Dot11Elt**, veja você mesmo:

``` python
>>> from scapy.all import *
>>> ls(Dot11Elt)
ID         : ByteEnumField                       = (0)
len        : FieldLenField                       = (None)
info       : StrLenField                         = (b'')
>>> 
```

o-O, Parece que que queremos filtrar, são as informações de cada AP, então tudo deve está dentro desse StrLenField do tipo bytes ai **info**, a resposta é sim, mas ao mesmo tempo não. 

![gato](/images/P0x01/gato_teorema.jpeg)

Para explicar isso melhor precisamos entrender How the thinks works, primeiro que existe uma heraquia de camada no protocolo 802.11, examente como você ta pensando **802.11 Layers Hierarchy, let's me show you:

``` sh
[RadioTap]
-[Dot11]
-- [Dot11<Frame Type>]
--- [Dot11Elt]
--- [Dot11Elt]
       ...
--- [Dot11Elt]
```

Dentro do pacote principal, **RadioTap**, temos outras camadas que contém outros tipos de dados todos codificado. Se estamos filtrando apenas esse tipo objeto, é porque ele contém informações importantes e uma dela são o **SSID** da rede que está sendo capturada. Quando StrLenField é null ou string vazia, isso me diz uma coisa que essa rede está com SSID oculto. vamos verificar isso também:

``` python
    essid = (
        pkt[Dot11Elt].info
        if "\x00".encode() not in pkt[Dot11Elt].info and pkt[Dot11Elt].info != ""
        else "Hidden SSID"
    )
``

Olhando para hieraquia vemos algum interessante, se **Dot11Elt** é um subcamada fields de um **Dot11**, isso quer dizer que temos mais informações pra analizar, mas o que temos de novo aqui agora é o tal do **Dot11** que na versão ele é um frame e um feilds layers, se não me engado essa coisas é montada parecido com TCP, quando pegamos um pacote raw lá em bytes. Vamos ver a implementação do Dot11Elt

``` python
  711 class Dot11Elt(Packet):
  712     __slots__ = ["info"]
  713     name = "802.11 Information Element"
  714     fields_desc = [ByteEnumField("ID", 0, _dot11_info_elts_ids),
  715                    FieldLenField("len", None, "info", "B"),
  716                    StrLenField("info", "", length_from=lambda x: x.len,
  717                                max_length=255)]
  718     show_indent = 0
  719 
  720     def mysummary(self):
  721         if self.ID == 0:
  722             ssid = repr(self.info)
  723             if ssid[:2] in ['b"', "b'"]:
  724                 ssid = ssid[1:]
  725             return "SSID=%s" % ssid, [Dot11]
  726         else:
  727             return ""
  728 
  729     registered_ies = {}
  730 
```

Parece que acetamos não é mesmo, vamos ver o que temos no Dot11:

``` python
>>> ls(Dot11)
subtype    : BitField (4 bits)                   = (0)
type       : BitEnumField (2 bits)               = (0)
proto      : BitField (2 bits)                   = (0)
FCfield    : FlagsField (8 bits)                 = (<Flag 0 ()>)
ID         : ShortField                          = (0)
addr1      : MACField                            = ('00:00:00:00:00:00')
addr2      : MACField (Cond)                     = ('00:00:00:00:00:00')
addr3      : MACField (Cond)                     = ('00:00:00:00:00:00')
SC         : LEShortField (Cond)                 = (0)
addr4      : MACField (Cond)                     = ('00:00:00:00:00:00')
```

Sem dúvidas chegamos onde queremos se tem **MACFields**, é porque tem informações importantes, vamos para documentação verificar o que são esses atributos.

Documentation do11: [dot11 packet](https://scapy.readthedocs.io/en/latest/api/scapy.layers.dot11.html#id2)

Os Campos são addr1, addr2, addr3, addr4, são macaddress que pode ser o mac do roteador que está emitindo aquele sinal em um determinado tipo de canal. Esses endereços mac não são padrão, isso mesmo, dependendo do tipo de configuração do **To DS** e **From DS** esses valores são trocados dentro do **Frame Control fields**, ou seja, dentro dot11. Dessa forma, no frame Dot11 não temos To DS e From DS nesse caso ambos campos são definidos como Zero. dessa forma, o campo addr3 fica definido como o BSSID da rede. Olhe so:

![imagem mac address](/images/P0x01/mac-address-01.png)


A imagem acima detalha muito bem esse comportamento, mas e o **addr2**, qual sua utilidade nessa configuração, o **TA=SA** mac original de quem mandou esse frame, então o **addr2** será o **cliente conectado aquela rede wireless**. Vamos provar que isso olhando o source do scapy implementation, dessa forma, podemos verificar o comportamento e como é decodificado esse frame. 

``` python
  496 class Dot11(Packet):
  497     name = "802.11"
  498     fields_desc = [
  499         BitField("subtype", 0, 4),
  500         BitEnumField("type", 0, 2, ["Management", "Control", "Data",
  501                                     "Reserved"]),
  502         BitField("proto", 0, 2),
  503         FlagsField("FCfield", 0, 8, ["to-DS", "from-DS", "MF", "retry",
  504                                      "pw-mgt", "MD", "protected", "order"]),
  505         ShortField("ID", 0),
  506         MACField("addr1", ETHER_ANY),
  507         ConditionalField(
  508             MACField("addr2", ETHER_ANY),
  509             lambda pkt: (pkt.type != 1 or
  510                          pkt.subtype in [0x8, 0x9, 0xa, 0xb, 0xe, 0xf]),
  511         ),
  512         ConditionalField(
  513             MACField("addr3", ETHER_ANY),
  514             lambda pkt: pkt.type in [0, 2],
  515         ),
  516         ConditionalField(LEShortField("SC", 0), lambda pkt: pkt.type != 1),
  517         ConditionalField(
  518             MACField("addr4", ETHER_ANY),
  519             lambda pkt: (pkt.type == 2 and
  520                          pkt.FCfield & 3 == 3),  # from-DS+to-DS
  521         )
  522     ]
```

Agora já demos saltar essa informação já que ela é importante pra gente.

``` python
    bssid = pkt[Dot11].addr3
    client = pkt[Dot11].addr2
```

A próxima informação que precisamos coletar é o CH **channel** ou a frequencia que esses dados estão sendo transmitdo. O canal no brasil vai de 1 a 11 se não me engano, eu não vou falar de 5Hz aqui, base é base vai mudar? vai, mas sabendo a base é so derivar (em cima e em baixo LOPITAU ), segura essa referência ai.

![referencia](/images/P0x01/referencia.jpg)

Sem mais delongas, existe muitas formas de capturar o CH, de um pacote Dot11Elt, uma dela é decodicar usando a camada do RadioTap, vamos ver ela no python.

``` python
>>> ls(RadioTap)
version    : ByteField                           = (0)
pad        : ByteField                           = (0)
len        : LEShortField                        = (None)
present    : FlagsField (32 bits)                = (None)
Ext        : PacketListField (Cond)              = ([])
mac_timestamp : _RadiotapReversePadField (Cond)     = (0)
Flags      : _RadiotapReversePadField (Cond)     = (None)
Rate       : _RadiotapReversePadField (Cond)     = (0)
ChannelFrequency : _RadiotapReversePadField (Cond)     = (0)
ChannelFlags : FlagsField (Cond) (16 bits)         = (None)
dBm_AntSignal : _RadiotapReversePadField (Cond)     = (-256)
dBm_AntNoise : _RadiotapReversePadField (Cond)     = (-256)
Lock_Quality : _RadiotapReversePadField (Cond)     = (0)
Antenna    : _RadiotapReversePadField (Cond)     = (0)
RXFlags    : _RadiotapReversePadField (Cond)     = (None)
TXFlags    : _RadiotapReversePadField (Cond)     = (None)
ChannelPlusFlags : _RadiotapReversePadField (Cond)     = (None)
ChannelPlusFrequency : LEShortField (Cond)                 = (0)
ChannelPlusNumber : ByteField (Cond)                    = (0)
knownMCS   : _RadiotapReversePadField (Cond)     = (None)
Ness_LSB   : BitField (Cond) (1 bit)             = (0)
STBC_streams : BitField (Cond) (2 bits)            = (0)
FEC_type   : BitEnumField (Cond) (1 bit)         = (0)
HT_format  : BitEnumField (Cond) (1 bit)         = (0)
guard_interval : BitEnumField (Cond) (1 bit)         = (0)
MCS_bandwidth : BitEnumField (Cond) (2 bits)        = (0)
MCS_index  : ByteField (Cond)                    = (0)
A_MPDU_ref : _RadiotapReversePadField (Cond)     = (0)
A_MPDU_flags : FlagsField (Cond) (32 bits)         = (None)
KnownVHT   : _RadiotapReversePadField (Cond)     = (None)
PresentVHT : FlagsField (Cond) (8 bits)          = (None)
VHT_bandwidth : ByteEnumField (Cond)                = (0)
mcs_nss    : StrFixedLenField (Cond)             = (0)
GroupID    : ByteField (Cond)                    = (0)
PartialAID : ShortField (Cond)                   = (0)
timestamp  : _RadiotapReversePadField (Cond)     = (0)
ts_accuracy : LEShortField (Cond)                 = (0)
ts_position : ByteField (Cond)                    = (0)
ts_flags   : ByteField (Cond)                    = (0)
he_data1   : _RadiotapReversePadField (Cond)     = (0)
he_data2   : ShortField (Cond)                   = (0)
he_data3   : ShortField (Cond)                   = (0)
he_data4   : ShortField (Cond)                   = (0)
he_data5   : ShortField (Cond)                   = (0)
he_data6   : ShortField (Cond)                   = (0)
hemu_flags1 : _RadiotapReversePadField (Cond)     = (0)
hemu_flags2 : LEShortField (Cond)                 = (0)
RU_channel1 : FieldListField (Cond)               = ([])
RU_channel2 : FieldListField (Cond)               = ([])
hemuou_per_user_1 : _RadiotapReversePadField (Cond)     = (32767)
hemuou_per_user_2 : LEShortField (Cond)                 = (63)
hemuou_per_user_position : ByteField (Cond)                    = (0)
hemuou_per_user_known : FlagsField (Cond) (16 bits)         = (<Flag 0 ()>)
lsig_data1 : _RadiotapReversePadField (Cond)     = (<Flag 0 ()>)
lsig_length : BitField (Cond) (12 bits)           = (0)
lsig_rate  : BitField (Cond) (4 bits)            = (0)
notdecoded : StrLenField                         = (b'')
```

Não se iluda jovem, tem muita coisa ai nessa classe que pode ser sugerido ou até mesmo ignorado, mas as inforções que queremos estão definidas. O que queremos é o CH, logo temos:

``` python
    channel = pkt[RadioTap].Channel
```
Uma outra forma de filtar essa informação é verificando o ID do packet dot11Elt, se o if for igual a 3 o campo **info** irá conter o valor do canal do AP. Veja só:

``` python
  636         while isinstance(p, Dot11Elt):
  637             if p.ID == 0:
  638                 summary["ssid"] = plain_str(p.info)
  639             elif p.ID == 3:
  640                 summary["channel"] = ord(p.info)
  641             elif isinstance(p, Dot11EltCountry):
  642                 summary["country"] = plain_str(p.country_string[:2])
  643                 country_descriptor_types = {
  644                     b"I": "Indoor",
  645                     b"O": "Outdoor",
  646                     b"X": "Non-country",
  647                     b"\xff": "Ignored"
  648                 }
  649                 summary["country_desc_type"] = country_descriptor_types.get(
  650                     p.country_string[-1:]
  651                 )
  652             elif isinstance(p, Dot11EltRates):
  653                 summary["rates"] = p.rates
  654             elif isinstance(p, Dot11EltRSN):

```

A linha 640 na implementação do **scapy**, temos um examplo de como pegar essa informação fazendo um loop no pacote capturado. Dessa forma, podemos conveter os 3 bits do pacotes do fields **info**.

``` python 
  while isinstance(p, Dot11Elt):
    if p.ID == 3:
      channel = ord(p.info)
```

``` 
Channel
Bit Number
3
Structure
u16 frequency, u16 flags
Required Alignment
2
Units
MHz, bitmap
Tx/Rx frequency in MHz, followed by flags.

Currently, the following flags are defined:
```

 Já que estamos fazendo um loop em todas as partes do pacote que é da camada dot11Elt, podemos extrair o restante dos dados que queremos. Vejamos na implementação do scapy.

 ``` python
  654             elif isinstance(p, Dot11EltRSN):
  655                 if p.akm_suites:
  656                     auth = akmsuite_types.get(p.akm_suites[0].suite)
  657                     crypto.add("WPA2/%s" % auth)
  658                 else:
  659                     crypto.add("WPA2")
  660             elif p.ID == 221:
  661                 if isinstance(p, Dot11EltMicrosoftWPA) or \
  662                         p.info.startswith(b'\x00P\xf2\x01\x01\x00'):
  663                     if p.akm_suites:
  664                         auth = akmsuite_types.get(p.akm_suites[0].suite)
  665                         crypto.add("WPA/%s" % auth)
  666                     else:
  667                         crypto.add("WPA")
  668             p = p.payload
  669         if not crypto:
  670             if self.cap.privacy:
  671                 crypto.add("WEP")
  672             else:
  673                 crypto.add("OPN")
  674         summary["crypto"] = crypto
  675         return summary
 ```

 Veja que na linha 668 a gente tem uma especie de ponteiro, que sempre a aponta para o próximo payload que pode ser de vários tipos depedendo do frame ta sendo transmitido. Então no nosso código tem que ser implementado isso também.

 ``` python
    p = pkt[Dot11Elt]
    while isinstance(p, Dot11Elt):
      if p.ID == 3:
        channel = ord(p.info)
      p = p.payload
 ```

 Mas podemos também já pegar o bomde e verificar os outros campos que como a criptografia da rede por exemplo, vamos lá 

  ``` python
  p = pkt[Dot11Elt]
  crypto = set()
  # O pacote Dot11Beaon contém um fields chamado cap, na verdade é um FlagFields
  # que se a rede for WEB ele vem enable por default.
  capability = p.sprintf(
            "{Dot11Beacon:%Dot11Beacon.cap%}\
                {Dot11ProbeResp:%Dot11ProbeResp.cap%}"
        )
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
 ```

Certo, já temos todas informações suficientes para montar nosso scanner similar ao airodump-ng, mas essas informações estão soltas até o momento, podemo usar o Sqlite (Banco de Dados) para amarzenar ou colocar em um dict() com a key o bssid da rede capturada. 

## Channel Hopping

Se você rodar esse código tera um problema, ele vai capturar aglumas redes mas no mesmo canal, se você sacar mais vez lá o **airdoump-ng** ele fica mudando o CH constantemente (tela superior esquerda). Isso tem um nome, geralmente chamado de **channel hopping**. Mas é simple, o nome só assusta, em resumo você precisa ficar mundando de cado e capturando redes ou sejá vai precisar de execução paralela, ou um **Process**, **Thread** ou de um **loop (for while)** se for preguiçoso. Eu partidulamente uso **Process**, em fim precisamos de rodar um commando para mudar o channel e mantar a placa no modo monitor.

``` python
  def channel_hopper(self, interface):
      while True:
          try:
              channel = randrange(1, 11)
              os.system("iw dev %s set channel %d" % (interface, channel))
              time.sleep(1)
          except KeyboardInterrupt:
              break
```
Com o comando **iw** podemos mudar o canal em que nossa placa está escutando por redes wireless, com isso aguardando 1 s e depois mudando de canal randomicamente, dessa forma podemos varrer todas redes disponíveis que estão emitindo em um determinado tipo de frenquência.

OBS:
Não vou colocar o código completo aqui, para excluir os **Script Kiddies** que estão lendo, nem sei se eles chegaram ate aqui.

![gato_kiddies](/images/P0x01/meme_gato.png)

## Conclusão 

Portanto, podemos concluir que saber o protocolo e seu **RFC** (Request for comments) ajudaria bastante fazer o que fizemos aqui, além da possibilidade de ler código fonte da ferramenta. Se você quer mais diversão que isso, veja essa mesma coisa em C eu prometo que você vai aprender batante, mesmo usado uma lib para facilitar o processo. Vai parecer chato o que vou dizer aqui, mas a base é tudo, talvez você não pegou a visão ainda mas certamente você vai lembrar desse artigo um dia quando encontrar alguma coisa que segue os mesmos princípios como tudo em cumputção. I hope so ter ajudado um curioso como eu aprender mais, no próximo episódio continuaremos com nossas descorbertas.

Nunca se esqueça, 
Keep Doing HACK THE PLANET 

Até um futuro próximo! 
Qualquer Dúvida ou Crítica Construtiva so mandar.

Livros que recomendo:
- Learn Python Network Programming 

Curso: https://www.udemy.com/course/construindo-security-tools-em-python/

Referências:

```
https://www.radiotap.org/
https://www.wireshark.org/docs/dfref/r/radiotap.html
https://www.wireshark.org/docs/
https://scapy.readthedocs.io/en/latest/api/scapy.layers.html
https://fossies.org/linux/scapy/scapy/layers/dot11.py
https://scapy.readthedocs.io/en/latest/api/scapy.layers.dot11.html#id2
Capitão América  
```