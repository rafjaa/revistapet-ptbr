--------------------------
Decorando código (Parte 1)
--------------------------

.. class:: endnote

+--------------------------------------------------+-------------------------------------------+
| .. image:: ../decorators/fabiangallina.jpg       |**Autor:** Fabián Ezequiel Gallina         |
|    :class: right foto                            |                                           |
|                                                  |**Tradução:** Rafael Alencar               |
+--------------------------------------------------+-------------------------------------------+

.. raw:: pdf

    Spacer 0 1cm

Neste artigo vou falar sobre como escrever decoradores (*decorators*)
em nossa linguagem favorita.

Um decorador é basicamente um *callable* [1]_ que envolve uma função ou
classe e que permite modificar o comportamento da mesma.
Isso pode parecer complicado à primeira vista, mas é muito mais fácil do
que parece.

Agora, sem mais delongas, vamos cozinhar alguns saborosos *callables*
revestidos caseiros.


Ingredientes
------------

* um *callable*
* um decorador
* café (opcional)
* açúcar (opcional)

Nosso *callable*, que será envolto, se parecerá mais ou menos com isto:

.. code-block:: python

    def yes(string='y', end='\n'):
        """Envia a string `string` para a saída padrão.
    
        Isso é similar ao que o comando do Unix `yes` faz.

        O valor padrão para `string` é 'y'.

        """

        print(string, sep='', end=end)

Nosso decorador, que será usado para envolver o *callable*, assemelha-se
a isto:

.. code-block:: python

    def log_callable(callable):
        """Decora callable e armazena informações sobre isso.

        Registra quantas vezes o callable foi chamado e os parâmetros que
        ele recebeu.

        """

        if not getattr(log_callable, 'count_dict', None):
            log_callable.count_dict = {}

        log_callable.count_dict.setdefault(
            callable.__name__, 0
        )

        def wrap(*args, **kwargs):
            callable(*args, **kwargs)
            log_callable.count_dict[callable.__name__] += 1
            mensagem = []

            mensagem.append(
                """Chamou: '{0}' '{1} vezes'""".format(
                    callable.__name__,
                    log_callable.count_dict[callable.__name__]
                )
            )
            mensagem.append(
                """Argumentos: {0}""".format(
                    ", ".join(map(str, args))
                )
            )
            mensagem.append(
                """Argumentos de palavra chave: {0}""".format(
                    ", ".join(["{0}={1}".format(key, value) \
                               for (key, value) in kwargs.items()])
                )
            )

            logging.debug("; ".join(mensagem))

        return wrap

O café serve apenas para você se manter acordado enquanto escreve código
tarde da noite, e o açúcar pode ser usado com o café, o que não é
obrigatório, já que todos sabem que o açúcar pode ser consumido a
colheradas.

Preparação
----------

Uma vez que temos nosso *callable* e nosso decorador, nós os
misturaremos em uma tigela.

Em Python nós possuímos 2 formas totalmente válidas para misturá-los.

A primeira, doce, com o açúcar sintático:

.. code-block:: python

    @log_callable
    def yes(string='y', end='\n'):
       [...]

A segunda [2]_, somente para os diabéticos:

.. code-block:: python

    yes = log_callable(yes)

Voilá, nosso *callable* agora está decorado.

Agora irei falar sobre a anatomia básica de um decorador para que o
nosso amigo verdureiro não possa nos enganar quando formos escolher um.

Com o que um decorador se parece
--------------------------------

Classes e funções podem ser decoradores. Estes decoradores podem ou não
receber argumentos (além dos argumento que eles recebem originalmente).

Então nós temos dois grandes grupos:

1) Funções decoradoras
   a) Sem argumentos
   b) Com argumentos

2) Classes decoradoras
   a) Sem argumentos
   b) Com argumentos

Um *callable* decorado *precisa* ser chamado explicitamente se o
programador quer que ele seja executado. Se isso não ocorrer o decorador
poderá impedir a execução do mesmo.

Exemplo:

.. code-block:: python

    def desativar(callable):
        """Decora um callable e impede sua execução."""

        def wrap(*args, **kwargs):
            logging.debug("{0} foi chamado, mas sua execução foi impedida.".format(
                callable.__name__)
            )

        return wrap

    @disable
    def yes(string='y', end='\n'):
       [...]

Funções decoradoras sem argumentos
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Em uma função decoradora que não recebe argumentos, seu primeiro e único
parâmetro é o *callable* a ser decorado. Na função aninhada é onde serão
recebidos os argumentos posicionais e de palavra chave (*keyword*).

Isto pode ser observado em qualquer um dos exemplos anteriores.

Funções decoradoras com argumentos
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Agora nós vamos ver um exemplo de uma função decoradora que recebe
argumentos. O exemplo irá se basear na nossa definição anterior de
*log_callable* e irá nos permitir especificar se queremos contar o
número de chamadas.

Exemplo de *log_callable* com argumentos:

.. code-block:: python

    def log_callable(do_count):

        if not getattr(log_callable, 'count_dict', None) and do_count:
            log_callable.count_dict = {}

        if do_count:
            log_callable.count_dict.setdefault(
                callable.__name__, 0
            )

        def wrap(callable):
            def inner_wrap(*args, **kwargs):
                callable(*args, **kwargs)

                mensagem = []

                if do_count:
                    log_callable.count_dict.setdefault(
                        callable.__name__, 0
                    )
                    log_callable.count_dict[callable.__name__] += 1
                    mensagem.append(
                        u"""Chamou: '{0}' '{1} vezes'""".format(
                            callable.__name__,
                            log_callable.count_dict[callable.__name__],
                        )
                    )
                else:
                    mensagem.append(u"""Chamou: '{0}'""".format(callable.__name__))

                mensagem.append(u"""Argumentos: {0}""".format(", ".join(args)))
                mensagem.append(
                    u"""Argumentos de palavra chave: {0}""".format(
                        ", ".join(["{0}={1}".format(key, value) \
                                   for (key, value) in kwargs.items()])
                    )
                )

                logging.debug("; ".join(mensagem))

            return inner_wrap

        return wrap

Uma função decoradora com argumentos recebe os parâmetros que são
passados explicitamente ao decorador. O *callable* é recebido pela
primeira função aninhada e finalmente os argumentos passados a ele são
recebidos pela próxima função aninhada (neste caso a função inner_wrap).

A forma se se utilizar este decorador seria:

.. code-block:: python

    @log_callable(False)
    def yes(string='y', end='\n'):
       [...]

Classes decoradoras sem argumentos
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Como dissemos antes, o decorador e o *callable* não precisam ser
funções, eles também podem ser classes.

Esta é uma versão do nosso *log_callable* na forma de classe (sem argumentos):

    class LogCallable(object):
        """Decora callable e armazena informações sobre isso.

        Registra quantas vezes o callable foi chamado e os parâmetros que
        ele recebeu.

        """

        def __init__(self, callable):
            self.callable = callable

            if not getattr(LogCallable, 'count_dict', None):
                LogCallable.count_dict = {}

            LogCallable.count_dict.setdefault(
                callable.__name__, 0
            )


        def __call__(self, *args, **kwargs):
            self.callable(*args, **kwargs)
            LogCallable.count_dict[self.callable.__name__] += 1

            mensagem = []

            mensagem.append(
                """Chamou: '{0}' '{1} vezes'""".format(
                    self.callable.__name__,
                    LogCallable.count_dict[self.callable.__name__]
                )
            )
            mensagem.append(
                """Argumentos: {0}""".format(
                    ", ".join(map(str, args))
                )
            )
            mensagem.append(
                """Argumentos de palavra chave: {0}""".format(
                    ", ".join(["{0}={1}".format(key, value) \
                               for (key, value) in kwargs.items()])
                )
            )

            logging.debug("; ".join(mensagem))

Em uma classe decoradora que não recebe argumentos, o primeiro parâmetro
do método __init__ é o *callable* a ser decorado. O método __call__
é quem recebe os argumentos do *callable* decorado.

A diferença mais interessante com a versão baseada em função é que
utilizando uma classe decoradora nós evitamos de utilizar uma função
aninhada.

A forma de se utilizar este decorador é a mesma utilizada com a função
decoradora sem argumentos:

.. code-block:: python

    @LogCallable
    def yes(string='y', end='\n'):
        [...]

Classes decoradoras com argumentos
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Entendendo bem os 3 casos de decoradores anteriores, é possível imaginar
como seria uma classe decoradora com argumentos:

Exemplo de *LogCallable* com argumentos:

.. code-block:: python

    class LogCallable(object):
        """Decora callable e armazena informações sobre isso.

        Registra quantas vezes o callable foi chamado e os parâmetros que
        ele recebeu.

        """

        def __init__(self, do_count):
            self.do_count = do_count

            if not getattr(LogCallable, 'count_dict', None) and do_count:
                LogCallable.count_dict = {}

        def __call__(self, callable):

            def wrap(*args, **kwargs):
                callable(*args, **kwargs)

                mensagem = []

                if self.do_count:
                    LogCallable.count_dict.setdefault(
                        callable.__name__, 0
                    )
                    LogCallable.count_dict[callable.__name__] += 1
                    mensagem.append(
                        u"""Chamou: '{0}' '{1} vezes'""".format(
                            callable.__name__,
                            LogCallable.count_dict[callable.__name__],
                        )
                    )
                else:
                    mensagem.append(u"""called: '{0}'""".format(callable.__name__))

                mensagem.append(
                    u"""Argumentos: {0}""".format(
                        ", ".join(map(str, args))
                    )
                )
                mensagem.append(
                    u"""Argumentos de palavra chave: {0}""".format(
                        ", ".join(["{0}={1}".format(key, value) \
                                   for (key, value) in kwargs.items()])
                    )
                )

                logging.debug("; ".join(mensagem))

            return wrap

Em uma classe decoradora com parâmetros, estes são passados para o
método __init__. O *callable* decorado é recebido pelo método __call__ e
os seus argumentos são recebidos pela função aninhada (chamada *wrap*
neste exemplo).

A forma de se utilizar este decorador é *exatamente* a mesma utilizada
com a função decoradora com parâmetros:

.. code-block:: python

    @LogCallable(False)
    def yes(string='y', end='\n'):
        [...]


Finalizando
-----------

Decoradores abrem um mundo de possibilidades permitindo-nos produzir
códigos mais simples e mais legíveis, sendo uma questão de analisar cada
caso de uso particularmente. Na nossa *provável* próxima parte deste
texto nós veremos mais exemplos práticos e daremos uma olhada nos
decoradores de classes (não confunda isso com classes decoradoras ;-)

.. [1] Nome de função ou classe (versão simplificada: se não adicionar parênteses não é executado :)
.. [2] A primeira é a forma recomendada, e a que você provavelmente escolherá, exceto se estiver decorando classes no Python < 2.6
