---
layout: post
title: Engenharia Reversa [Basic]
categories: [cracking]
tags: [cracking,windows,keygen]
fullview: true
comments: true
---
Hoje nós vamos entender como são construídos aqueles keygen que alimenta a pirataria, na verdade veremos um exemplo simples. Esse artigo vai ser apenas pra fins didático, até porque começar por engenharia reversa já com software de grande é loucura, então vamos começar com os mais simples é bom que já procuro da uma introdução a lin. assembly e os registradores.

É não vou explicar quem criou a linguagem ou deixou de criar, essas coisas não vai ajudar em nada, sei lá, melhor não vou começar falando que existe dois tipos sintaxe de assembly, a INTEL e a AT & T, lembando a explicação aqui vai apenas para este artigo porque vou fazer outro artigo falando sobre o mesmo mais detalhado. Cada arquitetura dos computadores tem sua linguagem e para começar vamos aprender um pouco sobre os registradores eles são fundamentais para o entendimento desse artigo, vamos trabalhar com windows INTEL, como falei o conhecimento que vou passar aqui sobre assembly vai ser o suficiente para entender o que vou demostrar futuramente. entao não fique de boa, ainda vou revisar melhor isso. outra coisa não vai ser preciso para esse artigo mas a Stack é muito importante quando falamos de assembly então vai uma preview:

A região denominada Stack ou pilha (mesma coisa), é responsável por salvar o endereço de sub-rotinas, passar argumentos (dados) para funções, armazenar variáveis locais e etc. Como o próprio nome sugere, o stack (pilha)funciona como uma pilha, onde você vai pondo dados lá.logo, o esquema seria o mesmo de uma pilha de livros, por exemplo.o ultimo livro que você por nessa pilha, será o primeiro a sair, o primeiro a ser pego, correto?. fugindo um pouco dos termos técnicos,imagine a stack como um clube de festa (meu deus..), e as pessoas que entraram nessa festa serão os dados.. caso, o cara que chegar por ultimo na festa quiser sair, ele terá que ser o primeiro a sair.. esse esquema é chamado de LIFO - Last in, first out (ultimo a entrar,primeiro a sair). dois comandos em asm são usados com relação a stack :PUSH (empurra) e POP (retira) dados, veremos isso melhor no capitulo sobre instruçoes em assembly

<h3>Entendendo Os Registradores:</h3>
<p>Os Registradores são blocos de memória que são usados para receber  e guardar dados,Existem vários tipos de registradores, cada um com uma função específica e cada grupo com sua finalidade. Eles serão apresentados abaixo.</p>
<table class="wiki-content-table">
<tr>
<th>Registrador</th>
<th>Nome</th>
<th>Descrição</th>
</tr>
<tr>
<td>EAX</td>
<td>Acumulador</td>
<td>Utilizado em operações aritméticas, acesso de portas de entrada e saída, transferência de dados, entre outros.</td>
</tr>
<tr>
<td>EBX</td>
<td>Base</td>
<td>Utilizado como ponteiro para acessar a memória, índice, e auxiliar de operações aritméticas efetuadas por EAX.</td>
</tr>
<tr>
<td>ECX</td>
<td>Contador</td>
<td>Sua principal finalidade é servir de contador em laços de repetição.</td>
</tr>
<tr>
<td>EDX</td>
<td>Dados</td>
<td>Usado em operações aritméticas juntamente com EAX (EDX recebe o resto da divisão e o produto da multiplicação), acesso de portas de entrada e saída, entre outros.</td>
</tr>
</table>
<p> há ta em ordem alfabética, não isso é apenas uma coincidência, é serio nem sei a ordem que eles ficam na memória, mas nem sempre foi assim esse 'E' de 'Eax' indica Extended ,extendido. isso tudo por causa do 32 bit's, antigamente os processadores de 16 bit eram, ax,bx,cx,dx e ainda tem outros, mas não vamos precisar para esse artigos. cada register desse possui 32 bits. eles possuem duas partes de 16 bits cada esses 16 bits são os 16 bits 'altos' e os 16 bits 'baixos'.os 16 bits mais altos possuem também, duas partes de 8 bits cada: a parte alta (high) e a parte baixa (low). para representar esse tipo de register, usamos o seu nome seguido de H para alto ou do L de low para baixo.ficando assim : al e ah, bl e bh, cl e ch, dl e dh. vou tentar mostrar abaixo com um desenho que é Pank  pra fazer.</p>
<pre class="code">
+-------------------------------------------------------------+
|                            EAX                              |
+-------------------------------------------------------------+
|                             |              AX               |
+-------------------------------------------------------------+
|                             |     AH       |       AL       |
+-------------------------------------------------------------+
</pre>
<p>Pronto, isso vai ficar claro quando o artigo de shellcode for escrito, lá precisamos mover valores(movl), para evitar null bytes, os bad char... Preciso explicar isso já que toquei no assunto :(. vamos supor que EAX recebe esse valor EAX= 0x00000000, pronto então AX = 0x0000 e a parte baixa (AH e AL) 0x00. é simples.  tá bom até dimais falar dos registradores já pra explicar o resto do artigo, vai ter algumas instruções como JMP (salto).Vamos entender melhor quando chegar lá... uma coisa que posso mostrar é como os valores são movidos por exemplo: seu eu quiser mover o valor 7  para EAX, ficaria assim: <a>MOV EAX,7</a> ou mover 10 = A em Hex , MOV EAX, 0xa . então é assim que funciona mover um valor para um registrador no caso do windows , já no linux AT & T, muda fica ao contrário: <a>MOV $0x7</a> ou MOV $0xa, EAX. é só inverter no linux, calma não se preocupe vamos aprender mais em outro artigo aqui: então vamos para o artigo em si, o objetivo é pegar a chave de um simples campo de register usando um debugger, precisamos abrir a aplicação é analisar o código em assembly, pode ser o OllyDBG ou qualquer tipo de Debugger que tiver ae eu vou usar o Immunity Debugger é a mesma coisa com o OllyDGB. Nosso Objetivo é crackear o programa usando o debugger, existe muita formas de fazer isso muitas pessoas vai logo em:</p>
<img src="/images/posts/Sem%20t%C3%ADtulo.png"/>
<p>Pega todas as string da aplicação, Isso pode ajudar, mas não nesse caso, mas primeiro vamos entender o programa que vamos trabalhar beleza... O programa Debugger é simples todos eles tem as mesmas abas pode tá em locais diferentes, mas fazem a mesma coisas vai uma imagem ae::</p>
<img src="/images/posts/debuggerImmunity.png">
<p>Disassembly: é a área onde fica o código do programa disassemblado,e os endereços de cada instrução, mesmo se o não for em assembly ele vai interpretar como assembly,ele interpreta os bytes que estão lá como assembly e mostra seu respectivos endereços.<br>
Registradores: há esse ai você já sabe né ? são os registradores aquele que falei pra vocês,Acumulador,base,contador e dados. toda vez que uma instrução é executada os registradores são mudados com o valores, mas há uma coisa ali existe mais registradores que eu mostrei pra vocês, é porque esses ae são os registradores Apontadores entenda:
ESI: Registrador de Índice fonte. Bastante usado em movimentações de blocos de instruções apontando para a próxima instrução.<br>
EDI: registrador de indice destino (index destination). Pode ser usado como registrador de offset's.<br>
EIP: registrador apontador de instrução (instruction pointer). Ele é o offset do segmento da próxima instrução a ser executada. Esse é um dos registres especiais e não podem ser acessados por qualquer instrução<br>
</p>
<p>DUMP: essa janela do dump pode ser configurada de várias forma tais como: float(ponto flutuantes),HEX(8 BITs),short, long,text e disassembly  é mais um auxilio, é pode ser usado em uma comparação sem precisar abrir outro Debugger.</p>
<p>Stack: eu espero que você já saiba o que ela faz como funciona já falei sobre ela, mas tem uma coisa a Pilha ela cresce pra cima, ou seja note que em baixo 000000000 então ela vai empilhando pra cima, os endereços diminui mas ela vai sempre cresce pra cima.beleza então estamos pronto para começar nossa brincadeira:</p>
<p>O arquivo que vamos estudar foi baixado do forum http://crackmes.de/, ele contém muitos desses pequenos aplicativos e outros ainda bem mais complexo, que serve para o aprendizado, a aplicação que irei mostrar pra você tem a versão 01,02,03,04 cara uma com grau de dificuldade eu escolhi a versão 02, ela é bem simples e serve muito bem para ter uma base de como funciona. O massa é que o fórum tem muitas pessoas disposta a ajudar tirar suas duvidas.</p>
<pre>
Requisitos: Um Debugger (olly ou Immunity)
software Target :<a href="/files/v2.exe" target="_blank">V2</a>
</pre>
<p>Agora faça o download os arquivo e vamos lá, primeiro quando você abri o Olly primeira vez, ele mostra um msg,se não me engano é pra usar Dll local ou dll do sistema, um amigo meu falou que é isso ele pede pra substituir a dll local pela do sistema, pronto Olly tem suas loucuras com um tempo você vai entender o que estou falando há vamos configurar ele em um outro artigo sei lá agora não precisa,baixou todos arquivos ? instalou ? bom bom memino(a)tudo certo agora abra o debugger e   no immunity é f3, ele abri ou você vai no menu 'file' e selecione o nosso arquivo V2.exe se usar o immunity vai ficar assim:</p>
<img src="/images/posts/Immunity.png">
<p>No Olly vai ser a mesma coisa, acho que o mesmo F7 para executar passo a passo, step by step. é o que você mais vai usar precisamos analisar o código então saber o que os registradores então fazendo é muito bom, nossa aplicação ainda não foi iniciada pois precisa carregar na memoria as instruções de programa vamos lá, vamos executar click em Run Program ou F9, no Immunity lembra da primeira Imagem que coloquei de procurar as string botão direito 'search for' procurar por 'all referenced text strings' procurar por todas referencias de string, nesso caso não podemos usa-la pois a aplicação target pode ter encodado, notem que ele mostra as informações quando nós colocamos um serial errado ele retorna a MessageBox("Sorry, wrong code"), desculpe código Errado, ou mostrar o trecho na area de dessasembly:</p>
<center><img src="/images/posts/analise%20disassembly.png"></center>
<p> outra coisa toda vez que precisamos chamar no caso uma API, precisamos colocar os argumento invertido ou seja se lá na documentação  tiver 1,2,3,4 precisamos colocar 4,3,2,1 já falei que a stack inverte, então é a mesma coisa inverte bytes a bytes. o call (Messagebox) vai dizer pro processador que vamos usar a API MessageBox de message depois push(empurra) para stack o primeiro argumento, depois o segundo que é a string 'Sorry wrong code', depois o terceiro nosso titulo da Messagebox, 'Fishing with DiLA v0.2' e por ultimo nosso 4 argumento que é 10 ou A, são a mesma coisa, esse vai indicar que tipo de Messabox vai ser. </p>
<pre class="code">
int WINAPI MessageBox(
 __in_opt HWND hWnd, // nosso 10
 __in_opt LPCTSTR lpText, // texto da msg
 __in_opt LPCTSTR lpCaption,  //titulo da caixa de messagem
 __in UINT uType  // tipo de msg  Informação,erro, e etc..
);
</pre>
<p>função da API MessageBox como falei são 4 argumento que são passado investido para stack, já expliquei cada um deles na imagem acima, lembre sempre investido, para você consultar as API vá no site da microsoft lógico kk msdn.microsoft.com e veja os argumento passados.</p>
<img src="/images/posts/breakpoint.png">
<p> primeiro quando marcamos um breakpoint em um endereço, assim que a instrução for executada ira parar naquela linha ignorando toda ação, faça o teste coloque o BP no mesmo endereço que coloquei e você não conseguirá inserir a chave para ativação, isso tudo por aplicação da uma pausa, e por que isso? as vezes precisamos parar em um determinado endereço para saber o que ta acontecendo e o que tem em cada registrador qual vai a próxima instrução a ser executa o EIP,e etc..., outra coisa se colocarmos um BP, e voltarmos a execução do programa vai ta lá no mesmo lugar :D, isso facilita muito tanto o Olly quanto o immunity faz isso pra nós, até agora não descobrimos nada de importante para saber a chave do programa, vamos entender o porque agora precisamos analisar mais um parte do programa os operadores lógicos, que são a chave para saber qual a key para esse artigo há e outra coisa para retirar um BP só apertar o mesmo F2:</p>
<center><img src="/images/posts/analisecompleta.png"></center>
<p> primeiro vamos entender o que está acontecendo: o programa armazena no registrador EBX o valor 922 em decimal por que em Hex = 39A,'http://www.hexdictionary.com/hex/39A/' e em binário = 1110011010, em fiz note que depois disso ele usa a instrução DEC,(decrementa 1 ) tem outra função que faz ao contrário INC(incrementa 1), o valor de EBX é iqual a 922 quando o programa usa DEC o valor de EBX passa para '921', eu ia mostrar como calcular 39A, transformar pra decimal, mas deu preguiça então usei um conversor, depois disso temos o valor de EBX = 921, a instrução CMP é o IF, o SE em programação de alto nivel, ela comparar se os dois valores o de EBX que é 921 e o de EAX que é o valor que nós digitamos no campo da chave, AHAHAAH descobrimos que o valor de EAX tem que ser = a 921 para que ele Salte para o endereço 00401068, onde vai ser chamado nossa Messagebox de sucesso isso mesmo :D conseguimos. há essa instrução <a>JE</a>, se a comparação acima for igual, se for igual ele pura para o endereço ou função apontada. :D por fim vamos fazer mais um teste colocando um breakpoint na função CMP e verificar o registrador Base o EBX pra ver se tamos certo mesmo.
 </p>
<img src="/images/posts/breakpoint2.png">
<p> Note que  o valor que coloquei no EAX = '7b', que em decimal é 123, ehehe, lógico como não é igual ele continua a execução  e cai na instrução messagebox erro 'Sorry', mas agora já sabemos que o valor é 921 vamos testar vamos ver se é isso mesmo:</p>
<img src="/images/posts/debuggerimmunityanalise.png">
<p> Assim que a instrução JE verifica que EAX = ABX ele pula ou salta para a instrução e assim chamando nosso messagebox desejado 'Sucess, Thanks you for playing ;)' é isso, outra coisa a instrução destrui.windows ou destruir a janela é nosso X.</p>
