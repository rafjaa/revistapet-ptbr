----------------------------------------
from gc import commonsense - Finish Him!
----------------------------------------

.. class:: endnote

+---------------------------------------------------------+-------------------------------------------+
| .. image:: ../hacking_python_s1/claudiofreire.jpg       |**Autor:** Claudio Freire                  |
|    :class: right foto                                   |                                           |
|                                                         |**Tradução:** Rafael Alencar               |
+---------------------------------------------------------+-------------------------------------------+

.. raw:: pdf

    Spacer 0 1cm

Não sei se todos, mas muitos de nós que usamos Python (e qualquer linguagem de alto nível, na verdade),
nos sentimos atraídos por suas elegantes abstrações. E não menos por aquele que abstrai o gerenciamento
de memória, mais conhecido como o ``garbage collector`` (coletor de lixo).
	
Esta coisa estranha, tão reverenciada quanto ignorada, diz a lenda, nos permite programar sem nos preocuparmos
com a memória. Não é necessário alocá-la, não é necessário liberá-la... o ``garbage collector`` se encarrega
de tudo isso.

E, como em toda lenda, há *alguma* verdade nisto.

Neste artigo vamos analisar os mitos e verdades sobre o gerenciamento automático de memória. Muito do que
vamos abordar se aplica a várias linguagens - todas aquelas que empregam algum tipo de gerenciamento
automatizado de memória - mas, é claro, vamos nos focar no tipo de gerenciamento de memória utilizado pelo
Python. E não qualquer sabor de Python, pois eles são muitos... O CPython.

Finalização
-----------

Fugindo um pouco do que é reservar e liberar bytes - pois antes de analisarmos os detalhes profundos e viscerais
do CPython devemos conhecer a superfície que o cobre -, vamos dar uma olhada em algo que tem sérias repercussões
no gerenciamento de memória e recursos em geral.

Se o leitor alguma vez já programou em linguagens orientadas a objetos (não necessariamente Python), ele deve 
saber bastante sobre construtores. Pequenas funçõezinhas que, bem, constroem instâncias de objetos de alguma
classe em particular.

Por exemplo:

.. code-block:: Pycon

    >>> class ClasseInutil:
    ...     def __init__(self, valor):
    ...         self.valor = valor
    ... 
    >>> objetoInutil = ClasseInutil(3)
    >>> objetoInutil.valor
    3

O mesmo leitor também irá se lembrar de alguma coisa menos comum em Python: destrutores. Destrutores (assim chamados
em muitas linguagens, mas também conhecidos por outros nomes também) são funções que são invocadas para *liberar*
recursos muitas vezes associados a uma instância. Por exemplo, se nossa ``ClasseInutil`` possuía como atributo um arquivo,
um socket, ou algo que necessite ser *"fechado"* ou *"liberado"*, nós iremos querer um *destrutor* que realize esta
ação quando a instância deixar de existir.
	
Isto se chama finalização, e em Python implementa-se desta maneira:

.. code-block:: Pycon

    >>> class ClasseInutil:
    ...     def __init__(self, arquivo):
    ...         print "abrindo"
    ...         self.valor = open(arquivo, "r")
    ...     def __del__(self):
    ...         print "fechando"
    ...         self.valor.close()
    ...
    >>> objetoInutil = ClasseInutil("arquivo.txt")
    abrindo
    >>> objetoInutil = None
    fechando

Outro leitor poderia dizer *Hum... interessante*. Eu não vou dizer o contrário.

E sim, a classe se chama inútil pois os objetos do tipo arquivo em Python já possuem seu próprio destrutor embutido,
que fecha o arquivo. Mas este é só um exemplo.

Vida e obra de uma classe
-------------------------

Agora vem *a* pergunta que deve ser feita. Quando é que uma instância deixa de existir? Em que momento ``__del__`` é
chamado?

Na maioria das linguagens de alto nível que gerenciam a memória para nós, a definição é vaga: em algum momento,
quando não houver mais nenhuma referência associada a ela.

Nesta pequena frase há um mundo de especificações vagas. O que é uma referência associada?
Quando exatamente? Imediatamente após as referências remanescentes serem desassociadas? Um minuto depois?
Uma hora depois? *Um dia depois?*

Como a primeira pergunta é complicada, nós vamos ver a próxima. E para a segunda, terceira, quarta, quinta e sexta
pergunta... bem... não há uma resposta precisa **a partir da especificação da linguagem**. A especificação é, assim,
vaga, e intencionalmente vaga.

A utilidade de uma especificação vaga (não deixando claro quando uma instância será finalizada) é, acredite ou não,
muito grande. Se não fosse por isto, Jython não existiria. Para aqueles que não conhecem o Jython, ele é uma
implementação da linguagem Python, porém feita em Java - pois ninguém disse que todas as implementações da linguagem
devem ser feitas em C, e também nada impede que isso seja feito.

Se a especificação tivesse dito que todos os objetos são finalizados imediatamente após ficarem sem referências aos
mesmos, uma implementação feita em Java seria incrivelmente menos eficiente. Isto ocorre pois este requerimento é muito
diferente dos requerimentos impostos no ``garbage collector`` do Java. Sendo inespecífica, a especificação do Python
permite que o Jython reutilize o ``garbage collector`` do Java - o que torna o Jython viável.

Se algum leitor já escreveu destrutores em Java, ele já percebeu o problema: Python, como uma linguagem, não nos dá
nenhuma garantia sobre quando nosso destrutor ``__del__`` irá ser executado, somente que em algum momento isto ocorrerá.
Hoje, amanhã, no dia seguinte... ou quando o computador for desligado. Seja lá quando for. A especificação não determina,
e nenhuma destas opções é boa para o Python.

Na verdade, é pior: a especificação atual do Python diz que não há nenhuma garantia que o destrutor sequer será chamado
para os objetos ativos quando o interpretador for finalizado. Ou seja, se eu chamar ``sys.exit(0)``, os objetos que estão
instanciados poderão ou não ser finalizados. Assim, não há sequer garantia que o destrutor será eventualmente chamado para
todos os casos.

Já o CPython, ao contrário do Jython, implementa um tipo de ``garbage collector`` que é muito mais imediato na detecção de
referências inalcançáveis - pelo menos na maioria dos casos. Isto faz com que os destrutores do Python pareçam mágicos,
imediatos, quase como os destrutores do C++. E esta é a razão pela qual os destrutores no CPython sejam dez vezes mais
úteis do que eles são em Java. Ou Jython.

Muitos programadores Python, equivocadamente, associam o CPython (uma implementação da linguagem Python, entre muitas),
como se fosse a linguagem Python em si. Infelizmente eu sou um deles. É muito conveniente, deve-se admitir. Então, se nós
estamos desenvolvendo nosso código com base nesta comodidade, vamos fazer isto conscientemente, sabendo o que realmente
estamos fazendo, e quais são as limitações disso.


Referências cíclicas
--------------------

Nossa classe inútil utiliza um destrutor que fecha o arquivo... uma coisa que é considerada incorreta em Python.
Por que? - muitas pessoas perguntam.

Então, vamos ver:

.. code-block:: Pycon

    >>> objetoInutil = ClasseInutil("arquivo.txt")
    abrindo
    >>> objetoInutil2 = ClasseInutil("arquivo.txt")
    abrindo
    >>> objetoInutil.circular = objetoInutil2
    >>> uselessObject2.circular = objetoInutil
    >>> objetoInutil = objetoInutil2 = None

Agora, um exercício para o leitor: pense sobre o que será exibido no console após a última sentença.
Não é incomum equivocar-se e dizer: *será exibido "fechando" duas vezes*. Errado. Vá em frente e experimente.

Para entendermos o que está acontecendo, digite no terminal: ``import gc ; gc.garbage``. Lá estarão nossas duas
instâncias da ``ClasseInutil``.

O que aconteceu? Nós iremos ver isso em detalhes em outro momento. A coisa importante para se relembrar aqui é
que destrutores não se dão bem com referências circulares. E há muitas, muitas maneiras de criarmos referências
circulares, e elas nem sempre são fáceis de se detectar, e são sempre mais difíceis de se livrar. ``gc.garbage``
será nosso melhor amigo quando suspeitarmos deste tipo de problema.

Ressuscitando objetos
--------------------

As pessoas não são as únicas que podem receber ressuscitação cardiopulmonar. Os objetos em Python também podem.
Sinceramente, nunca encontrei nenhuma utilidade nisto. Para absolutamente nada. Mas alguém deve ter pensado que
era útil, já que isto é parte da linguagem.

Se um destrutor, no processo de desalocação, criar uma *nova referência acessível por ele mesmo ao objeto*, a
desalocação é cancelada, e o objeto continua a existir.

Talvez isto seja útil para depuração, ou para fazer coisas malucas. Imaginemos um recurso que só deve ser
destruído na thread principal (o que não é absurdo, e acontece algumas vezes). O destrutor irá, então, chamar
pelo ``thread.get_ident()`` e comparar com a thread principal. Se ela não estiver executando na thread correta,
será enfileirada para ser destruída na thread principal. Após o enfileiramento, uma nova referência ao objeto
será criada, e o CPython a detectará. Isto é perfeitamente legal.

Isto também pode acontecer acidentalmente, e é o mais importante para ser lembrado, uma vez que duvido que muitos
leitores irão querer fazer isto de propósito. Então é muito importante não deixar uma referência para ``self``
escapar de um destrutor, ou vamos acabar em uma situação feia. Vazamentos de memória, recursos não finalizados,
exceções. Coisas feias.

Vamos ver precisamente um caso onde iremos nos safar disso, pois o Python por si só lida com isto à sua própria
maneira:

.. code-block:: Pycon

    >>> class ClasseInutil:
    ...     def __init__(self, arquivo):
    ...         print "abrindo"
    ...         self.valor = open(arquivo, "r")
    ...     def __del__(self):
    ...         raise RuntimeError, "Quero interromper tudo"
    ...
    >>> try:
    ...    x = ClasseInutil("arquivo.txt")
    ...    # faz coisas
    ...    x = None
    ... except:
    ...    pass
    ...
    abrindo
    Exception RuntimeError: RuntimeError("Quero interromper tudo",) 
          in <bound method ClaseInutil.__del__ 
          of <__main__.ClaseInutil instance at 0x7f2b2873e4d0>> ignored


A parte engraçada do código acima não é que ele estoura. Isso é óbvio, após nós dispararmos uma ``RuntimeError`` de forma
explícita.
A parte engraçada é o que ele **não estoura**.

Seria de se esperar que ele disparasse uma ``RuntimeError``, que seria capturada pela cláusula ``except``, e então seria
ignorada **silenciosamente**. Mas se isso acontecesse, a referência não desapareceria, pois quando a exceção é disparada,
uma referência para ``self`` é armazenada no ``Traceback`` da exceção. E quando saísse do bloco ``except`` ele iria tentar
destruir o objeto novamente, disparando uma outra exceção, a qual ressuscitaria o objeto novamente... e novamente, e
novamente. Uma diversão infinita.

    **Nota**: *Acontece que as exceções possuem uma referência a todas as variáveis locais de onde elas são disparadas,
    já que é útil para os depuradores, e que permite manter as instâncias vivas ou mesmo ressuscitá-las.*

Então o CPython, bem ciente do assunto, ignora as exceções que tentam tratar o destruidor. Se o destruidor não captura uma
exceção, ela não pode ser disparada. O que faz sentido se você pensar sobre isso, pois o código que chamou o destruidor o fez
de forma implícita, por puro acaso, e raramente se saberia como tratar a exceção.

Outra forma comum em que uma referência ao ``self`` escapa despercebidamente é quando se usa ``closures``. Expressões lambda
como ```lambda x: self.attribute + x` possuem uma referência implícita a self, e se essas expressões escaparem, o self também
escapará.


Gerenciadores de contexto
-------------------------

Concluindo, destrutores são úteis, confortáveis e difíceis de se prever. Eles têm que ser usados com cuidado, e sempre que
assumirmos que os destrutores são chamados imediatamente após uma instância deixar de possuir referências, nós estaremos
criando código que funcionará adequadamente apenas em CPython.

Para se fechar arquivos de forma confiável, o Python nos oferece uma ferramenta melhor, mais previsível e com suporte uniforme
em todas as implementações da linguagem: a declaração ``with``:

.. code-block:: Pycon

    >>> with open("arquivo.txt", "r") as f:
    ...     # faz alguma coisa
    ...     # não precisa chamar o método f.close(),
    ...     # ele será chamado automaticamente quando o fluxo do programa sair do bloco 'with'

Não abordaremos os detalhes da declaração ``with``, mas é importante mencionar que ela não substitui os destrutores. Apenas
substitui o uso que fizemos dos destrutores neste artigo, ou seja, para fechar arquivos. A declaração ``with`` possui muitos
outros usos, e eu os convido a investigá-los.

