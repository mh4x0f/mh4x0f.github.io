---
layout: post
title: Tecnicas Avançadas Shellcoding [asm]
categories: [Assembly]
tags: [shellcode,linux,asm,syscall]
fullview: true
comments: true
---
<p>Você já deve saber que criando Funções e JMP já ajuda no bypass dos Bad chars e armazenar nas parte baixa e altas do registrador ajuda também e como você sabe zerar os registradores antes de trabalha-los <a>xor eax.eax</a>. em fim, algumas técnicas podem ser usada para fazer isso, mas existe outros métodos de ser ainda mais. Usando algumas funções menos usadas podemos chegar em resultados excelentes do desenvolvimento de shellcodes vamos lá. Let's Goooo</p>
<h3>Multiplicação Mul</h3>
<p>à primeira vista, parecer banal, e é óbvio. No entanto, quando o difícil desafio de diminuir o shellcode, ele revela-se bastante útil. Primeiro algumas informações básicas sobre a própria instrução mul. mul realiza uma multiplicam sem sinal de dois inteiros. Leva apenas um operador, o outro é implicitamente especificada pelo registo eax%. Assim, uma instrução mul comum poderia ser algo como isto:</p>

{% highlight c %}
movl $0x0A,% eax
mul $0x0A
{% endhighlight %}
<p>Isto multiplica o valor armazenado em% eax pelo operando de mul, que neste caso seria de 10 * 10. O resultado é, então, implicitamente armazenados em EDX: EAX. O resultado é armazenado durante um período de dois registos, porque tem o potencial de ser consideravelmente maior do que o valor anterior, podendo exceder a capacidade de um único registo (isto é também como pontos flutuantes são armazenados em alguns casos). Tá idaí?</p>
<p>Então, agora vem a pergunta sempre importante. Como podemos usar esses atributos para a nossa vantagem ao escrever shellcode? Bem, vamos pensar por um segundo, a instrução leva apenas um operando, portanto, uma vez que é uma instrução muito comum, ele irá gerar apenas dois bytes em nosso shellcode final. Ele multiplica tudo o que é passado para ele pelo valor armazenado em% eax e armazena o valor em ambos %edx e %eax, substituindo completamente o conteúdo de ambos os registos, independentemente de saber se é necessário fazê-lo, a fim de armazenar o resultado da multiplicação. Vamos colocar em nossas matemático chapéus por um segundo, e considerar isso, o que é o único resultado possível de uma multiplicação por 0? A resposta, como você deve ter adivinhado, é 0 Eu acho que está na hora de algum código de exemplo, então aqui está.:
{% highlight c %}
xorl %ecx, %ecx
mul %ecx
{% endhighlight %}
O que é isso shellcode fazendo? Bem, o 0 está fora do registo ecx% usando a instrução xor, então agora sabemos que% ecx é 0. Então ele faz um ecx% mul, que, como acabamos de aprender, multiplica é operando pelo valor em% eax, e, em seguida, passa a armazenar o resultado dessa multiplicação em EDX: EAX. Assim, independentemente do conteúdo anterior do% eax,% eax agora deve ser 0. Porém isso não é tudo,% edx é 0'd agora também, porque, apesar de não ocorrer nenhum estouro, ainda substitui o% edx registrar com o bit de sinal ( mais à esquerda bit) de% eax. Usando esta técnica nós podemos zerar três registros em apenas três bytes,(Olha só que economia pense você pode comprar 3 por 1 isso que é promoção meu amigo 3 por 10 três balas por 10 centavos quando eu era guri era assim )enquanto que por qualquer outro método (que eu saiba) que teria levado pelo menos seis. ou seja, se você tem que ser mão de vaca ao estremo parceiro, essas técnicas eu vi em um fórum blackhat no inicio achei estranho pelo fato de envolver muito mais instruções  em asm depois fui percebendo o porque logo veio o resultado poucos bytes no shellcode.</p>
<h3>Método da Instrução  DIV </h3>
<p>Essa Instrução  lógico é bem semelhante a 'mul', precisa apena de um argumento que é nosso registrador que divide o mesmo. mul ele armazena o resultado da divisão em% eax. Mais uma vez, vamos exigir o lado matemático do nosso cérebro para descobrir como podemos tirar proveito desta instrução. Mas, primeiro, vamos pensar sobre o que é normalmente armazenado no registrador eax%. O registo eax% mantém o valor de retorno de funções e / ou syscalls. A maioria syscalls que são usados &#8203;&#8203;em shellcoding retornará -1 (em caso de falha) ou um valor positivo de algum tipo, só raramente se voltará 0 (embora ele ocorrer). Então, se nós sabemos que depois de uma syscall é realizada,% eax terá um valor diferente de zero, e que a instrução divl% eax vai dividir% eax por si só, e depois armazenar o resultado em% eax, podemos dizer que a execução de a instrução eax divl% após uma syscall vai colocar o valor 1 em% eax. Então ... como isso é aplicável a shellcoding? Bem, a sua é outra coisa importante que% eax é utilizado passar a syscall específico que você gostaria de chamar para int $ 0x80. Acontece que a syscall que corresponde ao valor 1 é exit (). Agora, para um exemplo:</p>
{% highlight c %}
xorl %ebx,% ebx
mul %ebx
push %edx
pushl $0x3268732f /bin/sh
pushl $0x6e69622f
mov  %esp,% ebx
push %edx
push %ebx
mov %esp,%ecx
movb $0xB, %al // execve () syscall,
//não retornar a menos que ele falhar, caso em que ele retorna -1
int $ 0x80 //syscall
//syscall exit() ExitProcess
divl %eax # -1 / -1 = 1 // Aqui a Magica acontece :D
int $0x80 //
{% endhighlight %}
<p>Agora, temos uma função de saída de 3 bytes,que economia não ? enquanto que antes era 5 bytes.No entanto, há um problema, e se um syscall não retornar 0? Bem na situação estranha em que isso poderia acontecer, você pode fazer muitas coisas diferentes, como inc% eax,% e não% eax qualquer coisa que vai fazer %eax não-zero. Algumas pessoas dizem que a saída de não são importantes em shellcode, porque seu código é executado independentemente de haver ou não sai limpo.(mas pense bem economizar bytes), Eles estão certos,(of course) também, se você realmente precisa para salvar 3 bytes para caber seu shellcode em algum lugar, o exit () não vale a pena manter. No entanto, quando o seu código faz acabamento, ele irá tentar executar o que estava após a última instrução, o que provavelmente irá produzir um SIGILL (instrução ilegal) que é um erro muito estranho, e será registrado pelo sistema. Assim, um exit () simplesmente adiciona uma camada extra de cautela para o explorar, de modo que mesmo se ele falhar ou você não pode limpar todos os logs, pelo menos esta parte de sua presença será clara. se ligou :D é assim que funciona!</p>
<h3>Conclusão:</h3>
<p> I Hope, que você use essas dicas para fazer algum maligno futuramente, como sempre quem faz shellcoding precisa deixar cada vez menor, o tamanho de bytes, por exemplo uma /bin/sh o menor que ja vi funcional é 24 bytes, tem de 18 mas não funciona muito bem.</p>
