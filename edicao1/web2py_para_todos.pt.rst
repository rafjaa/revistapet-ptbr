-----------------
Web2Py para Todos
-----------------
.. class:: endnote

+-------------------------------------------+------------------------------------------------+
| .. image:: ../web2py/carnet-reingart.jpg  |**Autor:** Mariano Reingart                     |
|    :class: right foto                     |                                                |
|    :width: 3cm                            |Desenvolvedor de software e professor.          |
|                                           |Entusiasta do Software Livre, apaixonado por    |
|                                           |Python, PostgreSQL e Web2Py.                    |
|                                           |                                                |
|                                           |**Blog:** http://reingart.blogspot.com          |
|                                           |                                                |
|                                           |**Empresa:** http://www.sistemasagiles.com.ar   |
|                                           |                                                |
|                                           |**Tradução:** Rafael Alencar                    |
+-------------------------------------------+------------------------------------------------+

.. raw:: pdf

        Spacer 0 1cm

Introdução ao Web2py
--------------------

Web2Py é um *framework* web muito fácil de usar e aprender. Ele foi desenvolvido para propostas educacionais, e inclui as últimas tecnologias integradas de uma forma simples e clara (MVC, ORM, templates, Javascript, AJAX, CSS, entre outras.) o que o torna uma solução totalmente funcional para o desenvolvimento de aplicações web 2.0 (todo o design e a programação são realizados dentro do navegador!).

Em nossa humilde opinião, Web2py torna o desenvolvimento web rápido, fácil e mais eficiente, mantendo o foco na lógica de negócios e não em temas técnicos triviais ou esotéricos. De um modo geral, suas principais funcionalidades são:

* Processo de instalação muito simples, praticamente sem configuração (tanto de forma independente quanto com o mod_wsgi).
* Intuitivo e com uma curva de aprendizado muito pequena, ideal para ser ensinado em sala de aula para iniciantes.
* Sua Camada de Abstração de Banco de Dados (*Database Abstraction Layer - DAL*) permite definir tabelas sem o uso de classes complexas (as quais depois podem ser estendidas  com campos virtuais, de forma semelhante a um ORM) e sua linguagem de consulta em Python é muito semelhante ao SQL, proporcionando um grande poder declarativo e flexibilidade.
* Estrutura sólida embutida, incluindo ajax, menus, formulários, *caching*, GAE, *web services* (JSON, XML_RPC, AMF, SOAP), agendamento de tarefas, etc. Seu design integrado limpo e seguro previne os problemas mais comuns associados ao desenvolvimento web.
* Altamente pythônico: modelos, visões e controladores simples, limpos, explícitos e dinâmicos, com uma linguagem de template programável com Python, auxílios em HTML e mapeamento bidirecional de URLs para padrões avançados.
* Sem complicações associadas à linha de comando, ele inclui um ambiente de desenvolvimento integrado e ferramentas de gerenciamento totalmente online com um editor de código e HTML, sistema de tickets de erro, upload de arquivos, etc.

Neste primeiro artigo nós iremos ver as principais funcionalidades e o processo de instalação do Web2py, e para os próximos artigos a intenção é mostrar em detalhes as funcionalidades da ferramenta: o modelo de dados (*model*), os controladores (*controllers*), as visões (*views*), autenticação, formulários, CRUD, AJAX, etc.

Instalando o Web2Py
-------------------

O Web2py possui pacotes para diversos sistemas operacionais, de forma que a instalação é muito simples, e sua filosofia de "baterias inclusas" faz com que praticamente não tenhamos que baixar ou instalar outras dependências (bibliotecas ou pacotes). 

Windows
~~~~~~~

Para o sistema operacional Windows, encontraremos um pacote compactado com tudo o que precisamos, bastando seguir as instruções para ter o Web2py funcionando:

* Faça o download do pacote tudo-em-um `web2py_win.zip <http://www.web2py.com/examples/static/web2py_win.zip>`_
* Descompacte-o
* Execute com um duplo clique o arquivo `web2py.exe`

Mac
~~~

O processo de instalação no Mac é muito semelhante ao do Windows, utilizando o pacote compactado `web2py_osx.zip <http://www.web2py.com/examples/static/web2py_osx.zip>`_.
Você precisa apenas descompactá-lo e executar o arquivo `web2py.app` para iniciar o programa.

GNU/Linux
~~~~~~~~~

No momento não há pacotes para diferentes distribuições do GNU/Linux, pois na maioria dos casos você pode simplesmente executar o Web2py a partir do código-fonte, uma vez que o Python e as principais dependências geralmente estão pré-instalados.

Para utilizar o Web2py a partir do código-fonte, siga estes passos:

* Instale as dependências (Python e os conectores para o banco de dados)
* Baixe o código-fonte em `web2py_src.zip <http://www.web2py.com/examples/static/web2py_src.zip>`_
* Descompacte-o
* Execute no terminal o comando `python web2py.py`

Como exemplo, no Ubuntu (ou Debian) abra o terminal e digite:

  sudo apt-get install python-psycopg2 
  wget http://www.web2py.com/examples/static/web2py_src.zip
  unzip web2py_src.zip
  cd web2py
  python web2py.py


Um breve passeio
----------------

Aqui faremos uma rápida demonstração dos principais recursos do Web2py. 

**Nota**: Os links funcionarão apenas se o Web2py estiver executando na máquina local, na porta 8000 (configuração padrão).

**Importante**: O idioma das páginas web é exibido de acordo com as configurações do navegador (disponível em: Inglês, Espanhol, Português, etc.).

Início
~~~~~~

Quando você iniciar o web2py, será exibida a tela inicial enquanto o programa é carregado:

.. figure:: http://www.web2py.com.ar/wiki/static/img/bienvenida.png
   :width: 55%

Logo aparecerá uma caixa de diálogo do servidor web de desenvolvimento embutido. Para iniciar o servidor escolha uma senha de administrador e pressione `start`:

.. figure:: http://www.web2py.com.ar/wiki/static/img/server.png
   :width: 33%

Bem-vindo
~~~~~~~~~

Quando o servidor inicializar, o Web2py irá abrir um navegador com a `página de boas vindas <http://127.0.0.1:8000/welcome/default/index>`_ padrão:

.. figure:: http://www.web2py.com.ar/wiki/static/img/welcome.png

Esta página é a aplicação padrão, um "esqueleto" que é utilizado quando nós criamos aplicações Web2Py.

Basicamente temos diversos links para a interface de administração, documentação, exemplos interativos e uma curta descrição sobre a página que você está visualizando:

* Você visualizou a URL `.../default/index`
* A qual chamou a função index() localizada no arquivo `.../controllers/default.py`
* A saída do arquivo é um dicionário que foi renderizado pela *view* `.../views/default/index.html`


Interface Administrativa
~~~~~~~~~~~~~~~~~~~~~~~~

Uma vez que temos o Web2py executando e podemos ver a página principal, podemos começar a criar e editar nossas aplicações web acessando a `interface administrativa <http://127.0.0.1:8000/admin/>`_:

.. figure:: http://www.web2py.com.ar/wiki/static/img/admin0.png

Nesta página, você precisará fornecer a senha definida no passo anterior. Será exibida uma listagem com as aplicações instaladas:

.. figure:: http://www.web2py.com.ar/wiki/static/img/admin1.png

Aqui nós podemos criar novas aplicações, realizar o upload ou download de aplicações já desenvolvidas, editar código-fonte e arquivos HTML, realizar o upload de arquivos estáticos, traduzir mensagens, revisar o log de erros, etc. Todos estes tópicos serão abordados nos próximos artigos.

Neste caso, nós iremos entrar na aplicação padrão `welcome <http://127.0.0.1:8000/admin/default/design/welcome>`_ , clicando no link *EDIT* (editar).

.. figure:: http://www.web2py.com.ar/wiki/static/img/admin2.png

E uma vez lá, nós podemos modificar o código-fonte do controlador (*controller*) principal (`default.py <http://127.0.0.1:8000/admin/default/edit/welcome/controllers/default.py>`_) clicando no link *edit* (editar):

.. figure:: http://www.web2py.com.ar/wiki/static/img/admin3.png

Os links acima nos permitem editar rapidamente os templates HTML relacionados ao projeto e testar as funcionalidades expostas.

Nós podemos ver que o código do programa de *hello world* é muito simples. Nós possuímos o `index` (que é executado automaticamente quando entramos na aplicação), o qual exibe a mensagem intermitente "Welcome to Web2py" e retorna um dicionário com a variável `message='Hello World'` que será utilizado para gerar a página web:

.. code-block:: python

  def index():
    """
    Ação de exemplo utilizando o operador de internacionalização T e mensagem
    flash renderizada por views/default/index.html ou views/generic.html
    """
    response.flash = T('Welcome to web2py')
    return dict(message=T('Hello World'))

Observe que este é todo o código que é utilizado na função para gerar a página web. Não é obrigatório executar scripts de gerenciamento, modificar arquivos de configuração, mapear URLs com expressões regulares e/ou importar vários módulos. Web2py cuida de todas estas questões para nós.

No momento vamos parar por aqui, já que a ideia deste artigo foi mostrar uma breve introdução à ferramenta. Nos próximos artigos nós iremos continuar com tópicos mais avançados.

Para os usuário que desejam continuar experimentando esta ferramenta, nós recomendamos seguir os exemplos interativos em: http://www.web2py.com.ar/examples/default/examples, onde são utilizados pequenos trechos de código para demonstrar os principais recursos do framework.

Resumo:
--------

Este artigo foi uma introdução ao *framework* Web2py, uma poderosa ferramenta para codificar sites de forma rápida e fácil. A ideia é aprofundar e expandir cada tópico em mais detalhes nos próximos artigos.

Na nossa opinião, ele é uma excelente escolha para começar com o desenvolvimento web sem perder de vista o desenvolvimento futuro de aplicações avançadas.

Como conselho, recomendamos assinar a `lista de discussão em português <http://groups.google.com/group/web2py-users-brazil>`_, onde você poderá consultar e analisar as últimas notícias e atualizações, já que o Web2py avança rápido, incluindo muitas funcionalidades novas em cada versão.

Recursos:
---------

* Site oficial: http://www.web2py.com/
* Grupo de usuários brasileiros no Google: http://groups.google.com/group/web2py-users-brazil
* Documentação principal (acesso gratuito ao livro publicado em HTML): http://www.web2py.com/book
* Referência rápida: `web2py-reference.pdf <http://bit.ly/a7fWSZ>`_

