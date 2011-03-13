--------------------------
Como esta revista é feita?
--------------------------

**Autor:** Roberto Alsina
**Tradução:** Rafael Alencar

Esta revista é um produto da `PyAr <http://python.org.ar>`_, o grupo de
desenvolvedores Python da Argentina. Ela é um projeto criado de programadores
para programadores (e não um projeto de designers gráficos para designers
gráficos), o que tem suas vantagens e desvantagens.

A desvantagem está a vista. Eu tive que cuidar do design visual. Peço desculpas
por qualquer sangramento ocular que lhe tenha produzido.

A vantagem é que um de nós (eu) já havia pego em um pedaço de código (criado por um
monte de outras pessoas) e o chutei até que tivesse algo parecido com uma
revista (`um livro <http://nomuerde.netmanagers.com.ar>`_).

Assim, nosso genes de programadores nos permitem ter uma infraestrutura
descentralizada para o desenvolvimento de revistas online, em um sistema
multiusuário, multifunção, com múltiplas saídas (PDF e HTML até agora) e
automático.

Automático como? Atualizando todo o design do site e do PDF com *um comando*.

Estas são algumas das ferramentas que nós utilizamos, todas softwares livres:

``git`` e ``gitosis``
    Uma ótima ferramenta para controle de versão e uma grande ferramenta
    para gerenciamento de repositório.

``restructured text``
    Um formato de marcação para documentos. Permite criar arquivos de texto
    simples e exportá-los para praticamente qualquer formato.

``rest2web``
    Converte nossos arquivos de texto em um website.

``rst2pdf``
    Cria PDFs a partir de arquivos no formato *restructured text*.

``make``
    Assegura que cada comando seja executado quando necessário.

``rsync``
    Garante que tudo vá para o servidor para que você possa ver.

Como esta é uma revista sobre programação, ela necessita de alguns requerimentos particulares.

Código
    É necessário exibir código-fonte. Rst2pdf possui suporte nativo para isso
    utilizando a diretiva ``code-block``, mas ela não faz parte do padrão do
    *restructured text*. Assim, tive que aplicar um patch no rest2pdf para
    utilizar este recurso.

    Por sorte esta diretiva é totalmente genérica, e funciona para HTML da mesma
    forma que para PDF. Isto é o que eu tive que acrescentar no início do arquivo
    ``r2w.py``:

.. code-block:: python

    from rst2pdf import pygments_code_block_directive
    from docutils.parsers.rst import directives
    directives.register_directive('code-block', \
        pygments_code_block_directive.code_block_directive)

Feedback
    Uma vez que a ideia é obter feedback, você precisa possuir um canal de comunicação
    para o mesmo. Comentários no site via Disqus.

Tipografia
    É difícil encontrar um conjunto de fontes boas, modernas e consistentes.
    Preciso de pelo menos negrito, itálico e negrito + itálico para o texto e o
    mesmo em uma variante monoespaçada.

    As únicas famílias que encontrei completas são as tipografias DejaVu e Vera.

HTML
    Eu não presto para HTML, então eu peguei um arquivo CSS chamado LSR emprestado 
    de http://rst2a.com.
    A tipografia é feita via Google Font APIs.
  
Servidor
    Não espero que haja muito tráfego. E mesmo que ocorra, não haverá problema:
    *este é um site em HTML estático*, e o servidor em que está hospedado é
    cortesia da `Net Managers SRL <http://netmanagers.com.ar>`_.
