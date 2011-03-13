------------------------------------------
Introdução aos Testes Unitários com Python
------------------------------------------

**Autor:** Tomás Zulberti

**Tradução:** Rafael Alencar


O que é um teste unitário e porque utilizá-lo?
----------------------------------------------

Testes unitários são aqueles onde cada parte (módulo, classe, função) do programa são testadas separadamente. Idealmente, você irá testar todas as suas funções e todos os casos possíveis para cada uma.

Testes unitários possuem diversas vantagens:

* Você pode testar se o programa funciona corretamente. Em Python, testes permitem que você identifique variáveis inexistentes ou os tipos esperados em uma função (em outras linguagens isto se faz em tempo de compilação).

* Você pode se assegurar que após uma mudança, todas as partes do programa continuarão funcionando corretamente. Isto inclui todas as partes que foram modificadas e todas aquelas que dependem delas. Isto é muito importante quando você faz parte de um time (com algum tipo de sistema de controle de versão).

* Os testes auto-documentam o código. Não diretamente, mas uma vez que os testes mostram o comportamento do código, lendo os testes você pode ver qual é a saída esperada para determinadas entradas. É claro que isto não quer dizer que você não precisa escrever documentação.

Então, por que existem bugs se nós podemos escrever testes? Pois os testes unitários possuem algumas desvantagens:

* Eles tomam tempo para serem escritos. Algumas classes são fáceis de se testar. Outras não.

* Quando você efetua grandes mudanças no código (refatoração), você precisa atualizar os testes. Quanto maior a mudança, maior é o trabalho de ajuste nos testes.

* Uma coisa muito importante: só por que os testes passaram, isso não quer dizer que o sistema funciona perfeitamente. Por exemplo, o CPython (o Python que você provavelmente está utilizando) possui inúmeros testes, e mesmo assim ainda existem bugs. Testes unitários apenas garantem um determinado funcionamento mínimo.


Como os testes devem ser feitos?
--------------------------------

Os testes precisam seguir estas regras:

* Testes devem rodar sem nenhuma ação humana. Isto quer dizer que nunca devem pedir para alguém digitar um valor. O teste por si só fornece todos os dados necessários.

* Testes precisam verificar o resultado da execução sem nenhuma interação. Novamente, para avaliar se o teste passou ou não, ele não deve te pedir para você decidir. Ou seja, ao escrever o teste, você precisa saber de antemão o resultado esperado para cada entrada fornecida.

* Testes devem ser independentes entre si. A saída de um teste não pode depender do resultado de um teste anterior.

Conhecendo estas regras, quais condições devem passar pelos testes?

* Condições que funcionem corretamente quando os valores de entrada forem válidos.

* Condições que falhem ou disparem uma exceção quando os valores de entrada sejam inválidos.

Se for possível, você deve começar a escrever os testes assim que começar a escrever os códigos, o que te permitirá:

* Identificar em detalhes o que o código que você está escrevendo deve fazer.

* Quando você escrever uma implementação que passe em todos os testes, você terá terminado. O que traz duas vantagens:

  * Você pode saber quando você deve parar de codificar.

  * Você não precisa codificar o que você não necessita.


Exemplo:
--------

Suponha que você possui os coeficientes de uma função quadrática e quer descobrir suas raízes. Ou seja, nós temos uma função desse tipo:

\[a * x^2 + b * x + c = 0\]
E queremos encontrar os valores \(r_{1}\) e \(r_{2}\) tais que:

\[a * r_{1}^2 + b * r_{1} + c = 0\]\[a * r_{2}^2 + b * r_{2} + c = 0\]
Além disso, dados os valores\(r_{1}\) e \(r_{2}\) nós queremos encontrar os valores de a, b e c. Nós sabemos que:

\[(x - r_{1}) * (x - r_{2}) = a x^2 + b x + c = 0\]
Tudo isso são conceitos de matemática que vimos na escola. Agora, vejamos um código que cumpre isso:

.. code-block:: Pycon

    import math

    class NaoEFuncaoQuadratica(Exception):
        pass

    class SemRaizesReais(Exception):
        pass

    def calcular_raizes(a, b, c):
        """ Dados os coeficientes a, b e c de uma função quadrática, encontra
        suas raízes. A função quadrática:
            ax**2 + b x + c = 0

        Irá retornar uma tupla com ambas as raízes onde a primeira raiz é menor
        ou igual a segunda.
        """
        if a == 0:
            raise NaoEFuncaoQuadratica()

        delta = b * b - 4 * a * c
        if delta < 0:
            raise SemRaizesReais()

        raiz_de_delta = math.sqrt(delta)
        primeira_raiz = (-1 * b + raiz_de_delta) / (2 * a)
        segunda_raiz = (-1 * b - raiz_de_delta) / (2 * a)

        # min e max são funções do Python
        menor = min(primeira_raiz, segunda_raiz)
        maior = max(primeira_raiz, segunda_raiz)
        return (menor, maior)


    def calcular_coeficientes(primeira_raiz, segunda_raiz):
        """Dadas as raízes de uma função quadrática, retorna os coeficientes.
        As funções quadráticas são definidas na forma:
            (x - r1) * (x - r2) = 0
        """
        # Você pode obter este resultado aplicando distribuição
        return (1, -1 * (primeira_raiz + segunda_raiz), primeira_raiz * segunda_raiz)

Finalmente, vamos ver os testes que nós escrevemos para o código anterior:

.. code-block:: Pycon

    import unittest
    from polynomial import calcular_raizes, calcular_coeficientes, \
                          NaoEFuncaoQuadratica, SemRaizesReais

    class TestPolinomial(unittest.TestCase):

        def test_calcular_raizes(self):
            RAIZES_COEFICIENTES = [
                ((1,0,0), (0, 0)),
                ((-1,1,2), (-1, 2)),
                ((-1,0,4), (-2, 2)),
            ]
            for coef, raizes_esperadas in RAIZES_COEFICIENTES:
                raizes = calcular_raizes(coef[0], coef[1], coef[2])
                self.assertEquals(raizes, raizes_esperadas)

        def test_formar_polinomio(self):
            RAIZES_COEFICIENTES = [
                ((1, 1), (1, -2, 1)),
                ((0, 0), (1, 0, 0)),
                ((2, -2), (1, 0, -4)),
                ((-4, 3), (1, 1, -12)),
            ]

            for raizes, coeficientes_esperados in RAIZES_COEFICIENTES:
                coeficientes = calcular_coeficientes(raizes[0], raizes[1])
                self.assertEquals(coeficientes, coeficientes_esperados)

        def test_nao_encontrou_raizes(self):
            self.assertRaises(SemRaizesReais, calcular_raizes, 1, 0, 4)

        def test_nao_quadratica(self):
            self.assertRaises(NaoEFuncaoQuadratica, calcular_raizes, 0, 2, 3)

        def test_integridade(self):
            raizes = [
                (0, 0),
                (2, 1),
                (2.5, 3.5),
                (100, 1000),
            ]
            for r1, r2 in raizes:
                a, b, c = calcular_coeficientes(r1[0], r2[1])
                raizes = calcular_raizes(a, b, c)
                self.assertEquals(raizes, (r1, r2))

        def test_integridade_falha(self):
            coeficientes = [
                (2, 3, 0),
                (-2, 0, 4),
                (2, 0, -4),
            ]
            for a, b, c in coeficientes:
                raizes = calcular_raizes(a, b, c)
                coeficientes = calcular_coeficientes(raizes[0], raizes[1])
                self.assertNotEqual(coeficientes, (a, b, c))

    if __name__ == '__main__':
        unittest.main()


É importante que os métodos e as classes dos nossos testes possuam a palavra ``test`` nos mesmos (com letra inicial maiúscula para as classes) para que elas possam ser identificados como classes utilizadas para testes.

Vamos ver passo a passo como o teste foi escrito:

1. Você importa o módulo ``unittest`` que faz parte o Python. Sempre que for fazer um teste você precisa criar uma classe, mesmo se o que for testar for uma função. Esta classe possui métodos chamados assertX, onde X varia. Você os utiliza para checar se o resultado está correto. Estes são os que eu mais uso:

**assertEqual(valor1, valor2)**
Checa se ambos valores são o mesmo, e falha se eles forem diferentes. Se eles são listas, é checado se todos os elementos das listas são iguais. O mesmo ocorre no caso de conjuntos.

**assertTrue(condição):**
Checa se a condição é verdadeira, e falha caso ela seja falsa.

**assertRaises(exceção, função, valor1, valor2, etc...):**
Checa se a exceção é disparada quando você chama a função com os argumentos valor1, valor2, etc...

2. Importa o código que você está testando, junto com as exceções que ele pode disparar.

3. Cria-se uma classe extendendo ``TestCase``, a qual irá herdar os métodos para os testes. Nestes métodos nós iremos utilizar assertEquals, etc. para checar se tudo está funcionando corretamente. Em nosso caso, nós definimos as funções:

**test_calcular_raizes**
Dada uma lista de coeficientes e suas raízes, testa se o resultado das raízes obtidas para o coeficiente são os mesmos que os esperados. Este teste checa se ``calcular_raizes`` funciona corretamente. Para fazer isso, nós iteramos por uma lista de duas tuplas:

* Uma tupla com os coeficientes para chamar a função.
* Uma tupla com as raízes esperadas para aqueles coeficientes. Estas raízes foram calculadas manualmente, e não utilizando o programa.

**test_formar_polinomio**
Dada uma lista de raízes, checa se os coeficientes estão corretos. Este teste checa se ``test_formar_polinomio`` funciona corretamente. Neste caso é utilizada uma lista com duas tuplas:

* A primeira com as raízes.
* A segunda com os coeficientes esperados para aquelas raízes.

**test_nao_encontrou_raizes**
Checa se ``calcular_raizes`` dispara uma exceção quando raízes reais não podem ser encontradas.

**test_nao_quadratica**
Checa o que acontece quando os coeficientes não fazem parte de uma função quadrática.

**test_integridade**
Dado um conjunto de raízes, calcula os coeficientes, e para cada coeficiente calcula as raízes novamente. As raízes obtidas devem ser as mesmas que as raízes iniciais.

**test_integridade_falha**
Checa o caso em que ocorre falha de integridade. Neste caso nós usamos funções cujo valor de *a* não é 1, então mesmo que as raízes sejam as mesmas, não é a mesma função quadrática.

4. Ao final do teste nós colocamos o código:

if __name__ == '__main__':
    unittest.main()

Assim, se nós executarmos python arquivo.py entraremos neste if, e executaremos todos os testes deste arquivo.

Suponhamos que estamos na pasta onde estão os arquivos:

pymag@localhost:/home/pymag$ ls
polinomial.py test_polinomial.py test_polinomial_falha.py

Então, vamos executar os testes:

pymag@localhost:/home/pymag$ python test_polinomial.py

    ......
    ----------------------------------------------------------------------
    Ran 6 tests in 0.006s

    OK


A saída mostra que todos os 6 testes das classe foram executados, e todos passaram. Agora nós iremos ver o que acontece quando um teste falha. Para isso, eu troquei um dos testes para que a segunda raiz seja menor que a primeira. Eu alterei o **test_integridade** para que uma das raízes esperadas seja (2, 1), quando na verdade deveria ser (1, 2), já que pela especificação a primeira raiz deve ser menor que a segunda. Esta alteração foi feita no arquivo test_polinomial_falha.py. Então, quando nós executamos os testes que falham, aparece:

pymag@localhost:/home/pymag$ python test_polinomial_falha.py
..F...
======================================================================
FAIL: test_integridade (test_polinomial.TestPolinomial)
----------------------------------------------------------------------
Traceback (most recent call last):
    File "/media/sdb5/svns/tzulberti/pymag/testing_01/source/test_polinomial.py",
    line 48, in test_integridade
    self.assertEquals(raizes, raizes_esperadas)
AssertionError: (1.0, 2.0) != (2, 1)

----------------------------------------------------------------------
Ran 6 tests in 0.006s

FAILED (failures=1)


Como podemos ver, todos os 6 testes foram executados novamente, dos quais um falhou. Também é indicado o erro e a linha onde ele ocorre.
