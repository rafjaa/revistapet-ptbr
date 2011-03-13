----------------------------------------------
Concorrência Indolor: O módulo multiprocessing
----------------------------------------------

.. class:: endnote

+-------------------------------------------+-------------------------------------------+
| .. image:: ../processing/yo-small.gif     |**Autor:** Roberto Alsina                  |
|    :class: right foto                     |                                           |
|                                           |O autor vem usando Python há algum tempo,  |
|                                           |e finalmente está pegando o jeito.         |
|                                           |                                           |
|                                           |**Blog:** http://lateral.netmanagers.com.ar|
|                                           |                                           |
|                                           |**twitter:** @ralsina                      |
|                                           |                                           |
|                                           |**identi.ca:** @ralsina                    |
|                                           |                                           |
|                                           |**Tradução:** Rafael Alencar               |
|                                           |                                           |
+-------------------------------------------+-------------------------------------------+

.. raw:: pdf

   Spacer 0 1cm

Às vezes você está trabalhando em um programa e se vê em um clássico
problema: a interface com o usuário trava. Está sendo efetuada uma
operação longa, e a janela "congela", trava, não se atualiza até que
a operação esteja concluída.

Algumas vezes nós podemos conviver com isso, mas geralmente isso passa
a imagem de amadorismo, ou de aplicações mal escritas.

A solução tradicional para este problema é fazer o seu programa utilizar
*multithreading*, e executar mais de uma tarefa paralelamente. Você passa
a operação demorada para uma thread secundária, faz o que tem que fazer
para que a aplicação se pareça "viva", espera até que a thread termine,
e segue adiante.

Aqui está um exemplo para brincarmos:

.. code-block:: Pycon

    # -*- coding: utf-8 -*-
    import threading
    import time

    def trabalhador():
        print "Começando a trabalhar"
        time.sleep(2)
        print "Trabalho concluído"

    def main():
        print "Iniciando o programa principal"
        thread = threading.Thread(target=trabalhador)
        print "Lançando a thread"
        thread.start()
        print "A thread foi lançada"
        
        # isAlive() retorna False quando a thread termina.
        while thread.isAlive():
            # Aqui ficará o código para que a aplicação
            # se pareça "viva"; pode ser uma barra de progresso,
            # ou simplesmente continuar trabalhando normalmente.
            print "A thread continua executando"
            # Esperamos um tempo, ao fim do qual a thread termina
            thread.join(.3)
        print "Programa finalizado"

    # Importante: os módulos não devem executar código
    # ao serem importados
    if __name__ == '__main__':
        main()


Que produz a saída:

    $ python demo_threading_1.py
    Iniciando o programa principal
    Lançando a thread
    A thread foi lançada
    A thread continua executando
    Começando a trabalhar
    A thread continua executando
    A thread continua executando
    A thread continua executando
    A thread continua executando
    A thread continua executando
    A thread continua executando
    Trabalho concluído
    Programa finalizado

É tentador dizer "threading é uma coisa linda!". Mas... lembre-se que este
foi um exemplo de brincadeira. Acontece que o uso de threads em Python possui
algumas ressalvas.

* Você não está utilizando múltiplos núcleos do processador.

  Como há um bloqueio global do interpretador, isto faz com que as instruções
  do Python, mesmo quando em mais de uma thread, sejam executadas em sequência.

  A exceção é se o seu programa executa muitas operações de entrada/saída, já
  que enquanto elas estão acontecendo, o interpretador continua funcionando.

* É fácil dar um tiro no próprio pé.

  Parafraseando Jamie Zawinsky, se quando você vê um problema e você pensa "Eu irei
  corrigi-lo usando threads"... agora você tem dois problemas.

* Não há nenhuma maneira de forçar uma thread a ser interrompida! E isto pode travar
  a aplicação de maneiras muito complicadas.

* É difícil depurar aplicações multithread, especificamente em condições de concorrência
  e deadlocks.

Então, o que nós podemos fazer? Utilizar processos ao invés de threads.
Vamos ver um exemplo que é suspeitamente familiar ao anterior:

.. code-block:: Pycon

    # -*- coding: utf-8 -*-
    import multiprocessing
    import time

    def trabalhador():
        print "Começando a trabalhar"
        time.sleep(2)
        print "Trabalho concluído"

    def main():
        print "Iniciando o programa principal"
        thread = processing.Process(target=trabalhador)
        print "Lançando a thread"
        thread.start()
        print "A thread foi lançada"
        
        # isAlive() retorna False quando a thread termina.
        while thread.isAlive():
            # Aqui ficará o código para que a aplicação
            # se pareça "viva"; pode ser uma barra de progresso,
            # ou simplesmente continuar trabalhando normalmente.
            print "A thread continua executando"
            # Esperamos um tempo, ao fim do qual a thread termina
            thread.join(.3)
        print "Programa finalizado"

    # Importante: os módulos não devem executar código
    # ao serem importados
    if __name__ == '__main__':
        main()

Sim, a única alteração é ``import multiprocessing`` ao invés de
``import threading`` e ``Process`` ao invés de ``Thread``. Agora, a função
``trabalhador`` é executada em um interpretador Python separado. Uma vez que
existem processos separados, eles irão utilizar os múltiplos núcleos que o seu
processador possuir, e o programa será muito mais rápido em um computador
moderno.

Eu mencionei deadlocks anteriormente. Você deve achar que com um pouco de cuidado, se você
colocar travas (*locks*) nas variáveis você pode evitar este problema. Bem, na verdade não.
Vamos ver duas funções ``f1`` e ``f2`` que utilizam duas variáveis ``x`` e ``y`` protegidas
pelas travas ``lockx`` e ``locky``.

.. code-block:: Pycon

    # -*- coding: utf-8 -*-
    import threading
    import time

    x = 4
    y = 6
    lock_x = threading.Lock()
    lock_y = threading.Lock()

    def f1():
        lock_x.acquire()
        time.sleep(2)
        lock_y.acquire()
        time.sleep(2)
        lock_x.release()
        lock_y.release()
        
    def f2():
        lock_y.acquire()
        time.sleep(2)
        lock_x.acquire()
        time.sleep(2)
        lock_y.release()
        lock_x.release()

    def main():
        print "Iniciando o programa principal"
        thread1 = threading.Thread(target=f1)
        thread2 = threading.Thread(target=f2)
        print "Lançando as threads"
        thread1.start()
        thread2.start()
        thread1.join()
        thread2.join()
        print "Threads finalizadas"
        print "Programa finalizado"

    # Importante: os módulos não devem executar código
    # ao serem importados
    if __name__ == '__main__':
        main()

Se você executar este programa, ele travará. Todas as variáveis estão protegidas com travas e ainda
assim continuam com problemas. O que acontece é que enquanto ``f1``` obtém ``x`` e espera por ``y``,
``f2`` adquiriu ``y`` e está esperando por ``x``. Uma vez que nenhuma função irá entregar à outra o que ela
precisa, ambas ficarão travadas.

Tentar depurar este tipo de coisa em programas não triviais é horrível, pois isso só acontece
em uma ordem determinada e com uma determinada variação de tempo. Pode ocorrer 100% do tempo em um
computador e nunca acontecer em outro computador que seja um pouco mais rápido (ou lento).

Adicione a isso que muitas estruturas de dados do Python (como dicionários) não são reentrantes, e você
precisa utilizar locks em muitas variáveis, e esses cenários são cada vez mais comuns.

E como isso funcionaria com o módulo ``multiprocessing``? Uma vez que você não está compartilhando recursos,
pois eles estão em processos separados, não ocorrem problemas com contenção de recursos, e também não
ocorrem deadlocks.

Quando você utiliza múltiplos processos, uma forma de gerenciá-los é passando os valores de que você
necessita como parâmetros. Suas funções então não terão "efeitos colaterais", de forma semelhante ao
paradigma de programação funcional, como em LISP ou Erlang. Exemplo:

.. code-block:: Pycon

    # -*- coding: utf-8 -*-
    import multiprocessing
    import time

    x = 4
    y = 6

    def f1(x,y):
        x = x+y
        print 'F1:', x
        
    def f2(x,y):
        y = x-y
        print 'F2:', y

    def main():
        print "Iniciando o programa principal"
        thread1 = processing.Process(target=f1, args=(x,y))
        thread2 = processing.Process(target=f2, args=(x,y))
        print "Lançando as threads"
        thread1.start()
        thread2.start()
        thread1.join()
        thread2.join()
        print "Threads finalizadas"
        print "Programa finalizado"
        print "X:",x,"Y:",y

    # Importante: os módulos não devem executar código
    # ao serem importados
    if __name__ == '__main__':
        main()

Por que eu não estou utilizando travas? Pois o ``x`` e o ``y`` de ``f1`` e ``f2`` não são os mesmos
do programa principal. Eles são cópias. Por que eu iria querer travar uma cópia?

Se existe uma situação onde um recurso precisa ser acessado sequencialmente, o módulo ``multiprocessing``
provê travas, semáforos, etc. com a mesma semântica do módulo ``threading``.

Ou você pode criar um processo para gerenciar aqueles recursos e passar seus dados por uma fila (classes
``Queue`` ou ``Pipe``) e voilà, o acesso agora é sequencial.

Em geral, com um pouco de cuidado no desenho do seu programa, o módulo ``multiprocessing`` traz todos os
benefícios do multiprocessamento, com o bônus de aproveitar melhor seu hardware, e evitar algumas dores
de cabeça.

Nota:
   O módulo ``multiprocessing`` está disponível na biblioteca padrão do Python a partir da versão 2.6.
   Para outras versões, você pode obter o módulo ``processing`` via PyPI.
