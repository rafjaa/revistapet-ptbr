------------------
Dinamismo Aplicado
------------------

.. class:: endnote

+-----------------------------------------------+-------------------------------------------+
| .. image:: ../dinamismo_aplicado/fisanotti.jpg|**Autor:** Juan Pedro Fisanotti            |
|    :class: right foto                         |                                           |
|                                               |**Tradução:** Rafael Alencar               |
+-----------------------------------------------+-------------------------------------------+

.. raw:: pdf

   Spacer 0 1cm

Muitas vezes acontece de eu comentar sobre alguma característica superior de uma linguagem com uma pessoa que não a conhece,
e esta pessoa reagir com frases do tipo "você usaria isso na vida real?" ou "muito legal, mas apenas na teoria, na prática
isso não funciona", etc...

Este é um comportamento normal, pois quando uma pessoa não está acostumada a pensar de uma forma diferente, ela não percebe facilmente a utilidade
de determinada funcionalidade. Estas não são grandes reflexões minhas, são ideias que Paul Graham descreveu muito bem `neste artigo
<http://paulgraham.com/avg.html>` (recomendado).

Mas o que posso mostrar, com minha humilde experiência, é um bom exemplo de como uma funcionalidade que possa soar "estranha",
quando bem aplicada, pode acabar se tornando "muito prática".

Pode haver centenas de coisas erradas no que eu digo, e seria muito bom se você me mostrasse algumas delas.


A funcionalidade "estranha"
---------------------------

Aqui é onde entra a funcionalidade "estranha" do Python. Ela soará estranha apenas para aqueles que nunca utilizaram este tipo de
comportamento da linguagem, é claro.

O recurso é a função ``getattr``: chamando-a, nós obtemos um atributo ou método de um determinado objeto. Por exemplo:

.. code-block:: python

    eu = "juan pedro"
    metodo = getattr(eu, "upper")
    metodo()   #isso retorna "JUAN PEDRO"

Explicando um pouco:

Com ``getattr(eu, "upper")`` nós efetivamente obtivemos o método ``upper`` do objeto ``eu``.
Atenção! Eu disse "obtivemos o método", e não "o resultado da chamada ao método". Estas são duas coisas bem diferentes.

Obter um método é como dar a ele um novo nome para chamá-lo mais tarde, como nós fizemos com ``metodo()``. ``metodo`` é um novo nome para aquele método em particular - o método ``upper`` do objeto ``eu``.

É importante mencionar que utilizar a variável (``metodo`` neste caso) é algo que eu fiz apenas para ser mais compreensível a primeira vista. O código anterior pode ser reescrito mais decentemente assim:

.. code-block:: python

    eu = "juan pedro"
    getattr(eu, "upper")()  # isso retorna "JUAN PEDRO"

Nós não armazenamos o método em uma variável, simplesmente o chamamos. Obtivemos o método e o chamamos, tudo na mesma linha.


O problema
----------

Nós possuímos uma classe **Foo**. Esta classe define 5 diferentes ações, representadas por 5 métodos: acao1, acao2, acao3, acao4, acao5.
A complexidade se dá pelo fato de que cada uma dessas ações são realizadas pela comunicação com um serviço, e há 4 serviços completamente diferentes pelos quais se podem realizar as ações: A, B, C e D.

Exemplo:
"Fazer a ação 1 no serviço B"
"Fazer a ação 3 no serviço D"
etc...

Na implementação, cada serviço redefine completamente o código executado para cada ação. Ou seja, o código para a ação 1 no serviço A é completamente diferente do código da ação 1 no serviço B, etc.

A classe **Foo** então precisa obter o nome do serviço como um parâmetro de cada ação, para que possa saber em qual serviço deve executá-la. Assim, poderíamos usar isso da seguinte maneira:

.. code-block:: python

    foo = Foo()   #nós criamos um novo objeto foo
    foo.acao1("A")  #nós chamamos a ação 1 no serviço A
    foo.acao1("C")  #nós chamamos a ação 1 no serviço C
    foo.acao3("B")  #nós chamamos a ação 3 no serviço B


Primeira solução "não dinâmica"
-------------------------------

Para muitos dos leitores, a primeira solução que virá à cabeça será que cada método (acaoX...) precisará possuir dentro de si um grande *if*, para cada serviço. Alguma coisa parecida com isso:

.. code-block:: python

    class Foo: 
      def acao1(self, servico): 
          if servico == "A": 
              #código para a ação 1 no serviço A
          elif servico == "B": 
              #código para a ação 1 no serviço B
          elif servico == "C": 
              #código para a ação 1 no serviço C
          elif servico == "D": 
              #código para a ação 1 no serviço D

Isso irá funcionar, eu não vou negar. Entretanto... do que eu não gostei nesta opção? Eu não gostei:

1) Aquele *if* será repetido em cada uma das ações, que são 5. Quando nós adicionarmos ou modificarmos serviços, precisaremos manter atualizado o mesmo *if* em todos os 5 métodos "acaoX".
2) O código se torna rapidamente ilegível quando se há muito o que fazer por cada ação.
3) Dá a sensação de que nós estamos misturando maçãs e laranjas, e que poderíamos ser mas organizados.
4) Estes *ifs* são duas linhas de código para cada ação e serviço, de modo que com 5 ações e 4 serviços, são 40 linhas de código apenas em *ifs*, não incluindo o código para as ações em si. São 40 linhas de código que não fazem o que nós queremos, e que nós precisamos apenas para decidir que código executar.


Melhorando a solução "não dinâmica"
-----------------------------------

Para as maçãs e laranjas e o problema de organização, a mais de um de vocês devem ter ocorrido ideias semelhantes esta:

.. code-block:: python

    class Foo: 
       def acao1(self, servico): 
           if servico == "A": 
               self.acao1_em_A()
           elif servico == "B":
               self.acao1_em_B()
           elif servico == "C":
               self.acao1_em_C()
           elif servico == "D":
               self.acao1_em_D()
    
       def acao1_em_A(self): 
               #código para a ação 1 no serviço A
     
       def acao1_em_B(self): 
               #código para a ação 1 no serviço B
     
       def acao1_em_C(self): 
               #código para a ação 1 no serviço C
    
       def acao1_em_D(self): 
               #código para a ação 1 no serviço D


Não posso negar, a separação em vários métodos ajuda um pouco na legibilidade e manutenibilidade. Considerando esta parte como resolvida, vamos nos esquecer dos métodos "acaoX_em_Y".
Mas ainda temos isso:

.. code-block:: python

       def acao1(self, servico): 
           if servico == "A": 
               self.acao1_em_A()
           elif servico == "B":
               self.acao1_em_B()
           elif servico == "C":
               self.acao1_em_C()
           elif servico == "D":
               self.acao1_em_D()

Continuo não gostando disso. Por quê?
Porque nós continuamos com o problema dos horríveis *ifs* espalhados por toda parte.
Minha opinião é que podemos fazer isso de uma forma melhor.


O estranho vem ao resgate: A solução dinâmica
---------------------------------------------

Bem, em teoria a coisa estranha deveria agora nos ajudar a resolver nosso problema. E como essa estranha funcionalidade do Python nos ajudará?
Vamos relembrar agora que nós decidimos esquecer de todos aqueles métodos "acaoX_em_Y", os quais nós aprovamos :). A parte feia do código é a escolha de qual método deve ser executado de acordo com o serviço fornecido.

Vamos ver então a versão "estranha" do código:

.. code-block:: python

    def acao1(self, servico): 
       getattr(self, "acao1_em_" + servico)()


Percebe o que está faltando? Não existem mais *ifs*!

Antes nós tínhamos 40 linhas de ifs, 8 linhas para cada ação, que apenas decidiam que código executar. Agora aquela decisão é feita com apenas uma linha de código por ação, o que resulta (com 5 ações) em um total de... 5 linhas!
5 linhas contra 40 equivale a 87% de código a menos.
Veja bem. A questão não é "ter poucas linhas de código é melhor". Neste caso, a vantagem é não ter código repetitivo e desnecessário para se manter.

E não apenas isso, nós também ganhamos uma outra vantagem muito importante: se nós adicionarmos ou removermos serviços amanhã, não será necessário alterar em nada o código que escolhe o método a ser executado. Precisaremos apenas adicionar as implementações (métodos acaoX_em_Y) e a classe saberá por si só como chamá-los, sem precisarmos fazer nenhuma mudança.
É bastante prático.


Conclusão
---------

Em um exemplo bastante simples, nós pudemos ver como uma funcionalidade "estranha" utilizada corretamente torna-se uma funcionalidade "prática".
E tome cuidado, pois quando você começa a usar estes recursos, torna-se muito tedioso voltar para as linguagens que não os possuem... É viciante, hehe.

PS: crédito a César Ballardini que me mostrou o artigo de Paul Graham's :D

